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


