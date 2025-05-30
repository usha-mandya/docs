---
title: Use containers for C++ development
linkTitle: Develop your app
weight: 20
keywords: C++, local, development
description: Learn how to develop your C++ application locally.
aliases:
  - /language/cpp/develop/
  - /guides/language/cpp/develop/
---

## Prerequisites

Complete [Containerize a C++ application](containerize.md).

## Overview

In this section, you'll learn how to set up a development environment for your containerized application. This includes:

- Configuring Compose to automatically update your running Compose services as you edit and save your code

## Get the sample application

Clone the sample application to use with this guide. Open a terminal, change directory to a directory that you want to work in, and run the following command to clone the repository:

```console
$ git clone https://github.com/dockersamples/c-plus-plus-docker.git && cd c-plus-plus-docker
```

## Automatically update services

Use Compose Watch to automatically update your running Compose services as you
edit and save your code. For more details about Compose Watch, see [Use Compose
Watch](/manuals/compose/how-tos/file-watch.md).

Open your `compose.yml` file in an IDE or text editor and then add the Compose Watch instructions. The following example shows how to add Compose Watch to your `compose.yml` file.

```yaml {hl_lines="11-14",linenos=true}
services:
  ok-api:
    image: ok-api
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    develop:
      watch:
        - action: rebuild
          path: .
```

Run the following command to run your application with Compose Watch.

```console
$ docker compose watch
```

Now, if you modify your `ok_api.cpp` you will see the changes in real time without re-building the image.

To test it out, open the `ok_api.cpp` file in your favorite text editor and change the message from `{"Status" : "OK"}` to `{"Status" : "Updated"}`. Save the file and refresh your browser at [http://localhost:8080](http://localhost:8080). You should see the updated message.

Press `ctrl+c` in the terminal to stop your application.

## Summary

In this section, you also learned how to use Compose Watch to automatically rebuild and run your container when you update your code.

Related information:

- [Compose file reference](/reference/compose-file/)
- [Compose file watch](/manuals/compose/how-tos/file-watch.md)
- [Multi-stage builds](/manuals/build/building/multi-stage.md)

## Next steps

In the next section, you'll take a look at how to set up a CI/CD pipeline using GitHub Actions.
