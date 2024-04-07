# `jq`

`jq` is a command-line JSON processor. TIL that it supports very continent pipe-like transformations called "filters", as well as a feature of joining JSON inputs. 

### Use case
I want to get a list of emails of commits authors that pushed to a repository in the recent year and compare it with a list of emails declared in Sentry. Basically, I intend to check who at our organization has to update their git config or emails at Sentry, to make "Suspect Commits" feature work.
#### filters
Sentry returns a list of users in this form:

```shell
$ curl --location "https://sentry.io/api/0/organizations/org/members/" --header "Authorization: Bearer <token>"
```

```json
[  
  {  
    "name": "John Doe",  
    "user": {  
      "emails": [  
        {  
          "email": "john@mail.com"
        },  
        {  
          "email": "doe@gmail.com"  
        }  
      ]
    }
  },  
  {  
    "name": "Jane Smith",  
    "user": {  
      "emails": [  
        {  
          "email": "jane@mail.com"
        }  
      ]
    }
  }
]
```

With `jq` we can transform this list to the list of all email with
```shell
$ echo <input_above> | jq "[.[] | .user.emails | .[] | .email ]"
---output
[
  "john@mail.com",
  "doe@gmail.com",
  "jane@mail.com"
]
```
what is very convenient.

#### `slurp` git input
We can use `--slurp` combined with `--raw-input` to load a string-per-line input and combine it to JSON array:
```shell
$ git log --pretty=format:"%ae" |
	sort | # to put repeated mails one after another
	uniq | # deletes repeated lines in a file
	jq --raw-input . |
	jq --slurp . # load input as JSON array of strings
```

#### subtract arrays
With `--slurp` we can also load those two arrays into one array, keeping references to where they came from (first or second input) and use it to subtract arrays
```shell
$ jq --slurp '.[0] - .[1]' \
 <(echo '["apple", "banana", "cherry", "date"]') \
 <(echo '["banana", "date"]')
--- output
[
  "apple",
  "cherry"
]
```

### The script
The final version of the script, using the mechanism described above, and a few additional tweaks, looks like this:
```shell
jq --slurp '.[0] - .[1]' \
<(git log --since="1 year ago" --pretty=format:"%ae" | grep -v "noreply" | sort | uniq | jq --raw-input . | jq --slurp . ) \
<(curl --location "https://sentry.io/api/0/organizations/<org>/members/" --header "Authorization: Bearer <token>" | jq "[.[] | .user.emails | .[] | .email ]")
```