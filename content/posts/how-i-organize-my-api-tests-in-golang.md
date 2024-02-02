---
author: "Velan Salis"
title: "How I organize my API tests in Golang"
date: 2022-07-04T13:02:23+05:30
description: ""
categories: []
tags: []
slug: "how-i-organize-my-api-tests-in-golang"
draft: false
---
Testing is apparently the most important part in writing API's since there could be so many ways the API's could mis-perform, and you need some way to check for their authenticity of response. Lately, I was tasked with writing unit tests for the API's in Golang at work. After a lot of trial and error, I figured out a way to organize my tests for better readability and maintainability. This is a quick and dirty rundown of my approach and, I'm glad if it helps you to organize your tests better :)

## Containment Structs
We need some structs that could act as a handy container for the tests that we are running. This organization is good since they provide a single place to access test methods, test data and also the runtime, dynamic data that needs to be persisted across tests. This might seem a bit overwhelming, but stay with me on this :) Here we assume the testing module name is module 
```
type module_test_payload struct {
    scenario string  method func(t *testing.T)
}

type module_test_config struct {
    tests []module_test_payload
}

type module_tests struct {
    test_api map[string]module_test_config
}
```
## Test Method Identifiers
The below const values act as a key for the test_api map so that the data and methods can be cleanly isolated and maintained inside the test_api map in module_tests. (One Identifier for one function)
```
const (
    GetSomething    string = "GetSomething"
    UpdateSomething string = "UpdateSomething"
    AddSomething    string = "AddSomething"
)

init()
```
With the below code snippet, we instantiate and populate the main struct i.e. module_tests in this case, so that it can be accessible to all the tests in the file and the data can be easily persisted across tests.
```
var (
    mt module_tests
)

func init() {
    mt = module_tests{}
    mt.test_api = make(map[string]module_test_config)
    mt.test_api[GetSomething] = module_test_config{
        tests: []module_test_payload{
            {
                scenario: "returns valid response",
                method:   mt._GetSomething_returns_valid_response,
            },
        }
    }
    st.test_api[UpdateSomething] = module_test_config{
        tests: []module_test_payload{
            {
                scenario: "updates valid response",
                method:   mt._UpdateSomething_returns_valid_response,
            },
        },
    }
}
```
In this step, we make a list of scenarios and the methods that we need to execute and pair them with the key identifier that we initialized earlier. This helps immensely in readability and gives a proper idea about what tests are we going to be running. Personally, This helps me in brainstorming the scenarios before I could write tests for methods. I just write scenarios and substitute the methods with blank functions and get a hang of how I'm going to be proceeding.

## Test Method
In the below code snippet, we loop over the methods we have attached with the identifier and execute them one by one.
(On a sidenote, if we are testing the method GetSomething, the test for that method will always be TestGetSomething. Golang will acknowledge this and only execute functions prefixed with Test)
```
func TestGetSomething(t *testing.T) {
    api := mt.test_api[GetSomething]
    testcases := api.tests
    
    t.Cleanup(func() {
        // Here we cleanup the tests.
    })
    
    for _, testcase := range testcases {
        t.Run(testcase.scenario, func(t *testing.T) {
            testcase.method(t)
        })
    }
}
```
Here we just need to write the method once and forget about it since we just need to add more methods to the tests array, and they will be executed in sequence. We can also set up a cleanup task in t.Cleanup(func(){}) so that the garbage that was created during the test can be cleaned up.
```
func (m *module_tests)_GetSomething_returns_valid_response(t *testing.T){
    // Write the testing code here.
}
```
The following code is the stub of the function that will be executed when the tests are running.

## Putting it all together
Although, It was minimal explanation from my end as to what this setup does, I encourage you to clone this code and check it out yourself. To put this all together, would span a lot of unwelcoming space in this page. Hence, I have created a code snippet in my repo for you to check it out. It's copy-pastable (new word, yay!)

## Why this approach?
Personally, I like the code to be readable and glanceable (Thanks [@matryer](https://github.com/matryer) for this word). I want to look at a piece of code and exactly understand what the code does. With this approach, we get the gist of what the code does in the init() function while we are writing the scenarios and the methods that the scenarios will use to further execute. 

This approach also helps in grouping the API functions that produce similar outputs without being tied down to writing all the functions under a Table Driven Test. I usually write the main scenarios in the init() and then run sub-scenarios that needs to be evaluated similarly, inside the stub-method in the good ol' table driven fashion. That greatly helps in debugging and maintenance.

Also, when there are a lots and lots of tests being added to the file, you will always have a link to those methods in the init() so that you can just find them all in one place. That saves me a lot of overhead, and I put that all the time in coding. It might seem a lot of prep work for just a bit of testing, but in a long run, I believe it's worth it.

## Wrap!
And, That's a wrap! Thanks for reading. I wrote this post in a hurry without a lot of brainstorming. I just wanted to put out the workflow I have deduced and which been working quite well for me. If this helped you, or if there's a way to make my code even better and less-work-proned (for the kind of lazy bug I am),  please do let me know here. Have a nice day.