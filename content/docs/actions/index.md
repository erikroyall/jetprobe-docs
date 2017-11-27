---
title: Actions
weight: 20
---

## Declaring Actions

An action in the context of a data pipeline is a stage or a phase that is part of the entire execution. Every action is separated by a dot (.) and takes an action builder
as a paramter. Let's see in detail, some of the inbuilt actions in JetProbe.

## Http Action

In case of http action, the parameter passed is the http request builder. For building a http request, a utility class known as `Http` is used. Here's a sample http request :

```scala
val getRequest = Http("get-post-detail")
    .get("https://${server.hostname}/posts/1")
```

Here's another POST request :

```scala
val postRequest = Http("create-a-user")
    .post("https://reqres.in/api/users")
    .body(
      fromFile("/path/to/post_req.json")
    )
```

Here `${server.hostname}` is a variable that can be interpolated based on the config passed during the execution of the test.

Once you have defined the request, then you can use http(...) helper method to include the above request as a part of the data pipline test.

```Scala
scenario("Http Test")
.http(getRequest)
.pause(1.seconds)
.http(postRequest)
```

Now you may also want to extract a some data from a request and use it in downstream processing.
Let's define another request, where we create a test user.

```scala
val postRequest = Http("create-a-new-user")
    .post("https://${api.server.address}/api/users")
    .body(
      fromFile("/path/to/post_req.json")
    ).header("Content-type", "application/json")
    .extract(
      jsonPath("$.name", saveAs = "user.name"),
      jsonPath("$.id", saveAs = "user.id")
    )
```

Once the values are saved in the variables, the runtime engine of Jetprobe would make these variables available to all the other downstream processing actions.

### Using runtime variables
```Scala
val getUser =  Http("get-user-details")
    .get("https://${server.hostname}/user/${user.id}")
```

Here `${user.id}` would be replaced by the value extracted by the json expression.

## Pause Action

A pause action, would simply pause the data pipeline, to allow the processing of data, after which the developer can execute the validations or some other set of actions.
The pause action takes a duration as an parameter.

**A Pause Action example**
```scala
scenario("A pause scenario").pause(4.seconds)
```

## SSH Action

A ssh action allows the developers to execute a remote command by providing the credentials for remote connections.

**SSH Action example**

```scala
//First define the ssh config
val sshConfig = SSHConfig(user = "admin", password = "secret", hostName = "xx.xx.xx.xx")

//Define the scenario
scenario("shell cmd scenario")
  .ssh(sshConfig, cmd = "cd /home/me/apps && mkdir temp && ls")
  .build
```

## Validation action

Validation actions take the `sink`, as a parameter which is nothing but the target that needs to be validated, and also the collection of validation rules that needs
to be executed for that particular sink.

A sink could be a database, a file, a messaging infrastructure or any http service. In the next section, we would see how to write validations for a given sink.