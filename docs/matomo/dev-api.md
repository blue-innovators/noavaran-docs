# How to generate Matomo API

## Getting started

To summarize the things you have to do to get setup:

*Install Matomo (for instance via git).

*Activate the developer mode:

```sh
./console development:enable
```

*Generate a plugin:

```sh
./console generate:plugin --name="MyApiPlugin"
```

There should now be a folder plugins/MyApiPlugin.

*activate the created plugin:

```sh
./console plugin:activate "MyApiPlugin"
```

## Let’s start creating an API

```sh
./console generate:api
```

Now, check these api links:

index.php?module=API&method=MyApiPlugin.getAnswerToLife

index.php?module=API&method=MyApiPlugin.getExampleReport&idSite=1&period=week&date=today&wonderful=1

## JSON response

for json response, you should and &format=json to end of api request


## API Call

1.Create a Token:

Create a token here: Settings > Personal > Security > Generate New Token

2.Type Of Request:

API Request should be POST and body should contains token_auth and a generated token

3.Body Of Request:

Just in case anyone else, like me, ends up here from searching for solutions to this problem:

Set your Content-Type to application/x-www-form-urlencoded. This is default for curl but not when using fetch() in the browser for example, which is why all the curl examples work but your code doesn’t



