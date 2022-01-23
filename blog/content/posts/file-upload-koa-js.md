---
author: "Velan Salis"
title: "Simple file upload using Koa.js (or Node.js in General)"
date: "2018-01-20"
description: "A post about how to use Koa.js to upload files"
categories: ["Tutorial"]
tags: ["koajs", "nodejs"]
slug: "file-upload-koa-js"
---

I have been using Koa.js for a while now and it turns out to be great. I’m having a lot of fun playing around with it since its simple and configurable. And I was searching for a _ground up_ way to execute file uploads so that I can tweak its functionality however I want to. And it turns out, It’s not as difficult as I thought. Before going forward, I’m assuming you have a basic knowledge of how to setup basic Koa.js server with routers since it’s being covered in so many articles. So I’ll be skipping the server setup with routers. [Click here](https://medium.com/@adam_bickford/creating-a-basic-site-with-koa-pt-1-f3e1711f7a9) if you want to go through an article that covers the same. So, Lets get coding. 😉

### Server Page : app.js

```
// app.js
const Koa = require("koa");
const koaBody = require("koa-body");
const logger = require("koa-logger");

const router = require("./router");

const app = new Koa();

app.use(koaBody(
    { multipart: true }
));

app.use(logger());

app.use(router.routes()).use(router.allowedMethods());

app.listen(3000, () => {
    console.log("Listening at 3000");
});
```

The above app.js file starts up the server and starts listening in port 3000 for requests. Simple stuff, nothing fancy. But things to be noted are the koa-body module and `{ multipart : true }` option. This part is really important, since koa-body parses the request body and populates Koa `ctx` object based on the form data being sent. When a file is being sent from the front-end, koa-body parses the request and the uploaded file attributes will be available under `ctx.request.files` which we can then access across Koa server to implement file upload functionality

### Router and Upload Logic : Router.js

_Well, Its not a good practice to add the server logic under router.js file since router is only used for routing the requests. You will have a controller file to take care of the logic when the request is being routed from router. But for the course of this tutorial, I have written the logic in the router itself._

In this part, you will need a package called promisepipe which I’ll explain in a second. For now, install the package by running the following command. And create a folder named uploads in the root of the project which will hold uploaded images.

```
npm install promisepipe
```

```
// router.js
const Koa = require("koa");
const Router = require("koa-router");
const promisePipe = require("promisepipe");
const fs = require("fs");
const path = require("path");

const router = new Router();

router.get("/", (context, next) => {
    context.body = "Hey";
});

router.post("/upload", async (context, next) {
    try {
        const uploadfile = context.request.files.file;
        const savefile = `${Date.now()}#${uploadfile.name}`;
        const readStream = fs.createReadStream(uploadfile.path);
        const writeStream = fs.createWriteStream(path.join("uploads", savefile));
        await promisePipe(
            readStream.on("error", () => {
                throw new Error({
                    errors: "File Read Error"
                });
            }),
            writeStream.on("error", () => {
                throw new Error({
                    errors: "Write Error"
                });
            })
        );
        context.body = {
            message: "File Uploaded"
        };
    } catch (err) {
        console.log(err);
        context.body = {
            message: "There was an error",
            errors: err
        };
    }
});

module.exports = router;
```

### The Upload Logic

Anything that is written in the bold letters in router.js is important and I’ll be explaining the mode of execution one by one.

- **/upload :** This is the POST route handler that contains the logic of what needs to be done when the front-end makes an upload request with a file attached to its body. When the URL is hit, the function gets executed.

- **uploadfile** : File attributes from parsed context.request.files.file in uploadfile stored in this variable so that its easier to access it later on in the code. In simple words, uploadfile contains the reference to the file to be uploaded to the server.

- **savefile** : The file name to be stored in the folder has to be unique since the same name will be stored in the database to trace down the files from the folder later. So we use `{Date.now()}#${uploadfile.name`to create unique name *(where *uploadfile.name* is the name of the file to be uploaded)*. Which will give us a string of 1560234246152#anyfile.txt for example. Since we add the timstamp, filename stays unique and it will be stored in savefile variable.

- **readStream/writestream** : Stream is nothing but a flow of bits from one end to another end. In here, we create a read stream from user’s local computer and write stream to server’s uploads directory. We pass uploadfile.path which contains user’s local file’s path and uploads folder to write stream. That sets up the stream needed for the upload.

- **promisePipe** : promisePipe is a package that takes in two streams ( read / write ) and starts reading from read file to the write file. In here, bits from the user’s local computer file will be written in streams to the uploads directory on the server with the name mentioned in the variable savefile . The best thing about promisepipe is it returns a promise. It returns a resolved response when the uploading is done or returns the rejected response when there is an error. So we can await untill the uploading is done and continue executing code thereafter, or we can use try/catch block to catch any errors while uploading and handle it.

By the end of execution, file will be saved in uploads folder and its name will be stored in savefile variable which can be added to the database for tracking down the files when they are requested. And if there is an error while reading or writing, it will be caught by the catch block and valid response to the front-end will be sent.

### Summing it all up

Koa.js is an amazing lightweight configurable framework for Node.js. And arguably, There could be tons of better ways to achieve the same results but this turned out to be the best one for me where i can configure the functionality from the ground up. So if you know of any way to make this code better with added functionalities, I’d look forward to hear it from you. Happy coding. Have a nice day 😄

### Important Links

- **Koa.js** :
  Koa is a new web framework designed by the team behind Express, which aims to be a smaller, more expressive, and more. https://koajs.com/
- **koa-body** :
  A Koa body parser middleware. Supports multipart, urlencoded and JSON request bodies. https://www.npmjs.com/package/koa-body

- **promisepipe** :
  Pipe node.js streams safely with Promises. https://www.npmjs.com/package/promisepipe