# Removing quotes from JSON strings with JQ

I have a project where I want to use the version in a `package.json` file in a Makefile, and I don't want to track that version in two places. I used `jq` to read the file and get one field, but I was getting a quoted string, and I needed to take off those quotes.

[This StackOverflow post had the answer](https://stackoverflow.com/questions/44656515/how-to-remove-double-quotes-in-jq-output-for-parsing-json-files-in-bash):

```sh
cat package.json | jq -r '.version'
```

That feeds `package.json` into `jq` and reads the `version` field. The `-r` (or `--raw-output`) option makes it output a raw string, not a quoted string literal.

Inside the Makefile, I had to use the `shell` function to get it to work, like this:

```makefile
TAG=$(shell cat package.json | jq -r '.version')
```
