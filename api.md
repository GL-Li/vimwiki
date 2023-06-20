## Outline
    - Major reference
    - Workflow and SOP
    - Minimal examples
    - Glossaries
    - Raw notes

## Major references

- API and Web Service Introduction [Udemy](https://www.udemy.com/course/api-and-web-service-introduction/)

## Workflow and SOP

## Minimal example

## Glossaries

################################################################################
=================================== raw notes =================================
################################################################################
## 2023-04-02 Sun

- [x] API: what is API - from I (interface) tell P (program) to run A (application)

    - Simple API
        - A: application, software that does a task, for exemple, google search
        - P: programming, program that run the task at the server side, for exemple, the code and infrastructure for google search.
        - I: interface, the user interface, for exemple, the google search webpage where users can type keywords and see the results.

    - complex API without web browser
        - this is the API developers talk about
        - the interface is at the server
        - cheap to crate by the application provider
        - flexible for users to programm the output

    - request, program, response

        - In the case of simple API when we search "tuna" on Google, the request is sent to a computer at `https"//google.com` and then locate a program called "search" and add "tuna" as a parameter `?q` for the program. The complete command is https://www.google.com/search?q=tuna`. The response is then sent back to the web browser. Other parameters are optional in real Google search.

- [ ] API: intro to HTTP for request and response

    - elements
        - start line,
            - version
            - method
            - parameters
            - status
        - headers
            - host
            - token
            - cookie
        - blank line to separate header and body
        - body

    - start line
        - request example: search tuna on Google. the actual request is (not google.com in it):
            ```
            GET /search?q=tuna HTTP/1.1
            ```

    - headers to specify
        - language
        - authorization
        - cache
        - server: https://google.com
        - content type to be sent or received in body
        - many others

    - body
        - content whose type is specified in headers

## 2023-04-03 Mon

- [x] API: XML and JSON for data type

    - XML
        - extensible markup language
        - any tags can be used, which only describe the content. While HTML each tag has a special meaning.
            ```xml
            <Pizza>
                <Size>small>/Size>
                <Toppings>
                    <Topping>onions</Topping>
                    <Topping>Mushroons</Topping>
                </Toppings>
            </Pizza>
            ```
        - can be saved as a xml file and opened with a browser.

    - JSON
        - JavaScript Object Notation
        - same exemple in json
            ```json
            {"Pizza" : [
                {"Size" : "small",
                 "Toppings" : ["onions", "mushrooms"]
                },

                {"Size" : "large",
                 "Toppings" : ["cheese", "meatball"]
                }
            ]}
            ```
        - save as a json file and open in a browser.

- [x] API: SOAP, outdated HTTP request/response format
    - Simple Object Access Protocol
    - replaced by REST, do not spend too much time on it.

## 2023-04-04 Tues

- [x] API: REST

    - Unlike SOAP, REST uses methods GET, POST, PUT, DELETE, ... in HTTP requests. SOAP has a placeholder POST for method but never used. Instead POST follows a protocal to achieve what can be done by the methods.
    - REST uses cache to store response in the memory of client computer after GET request

## 2023-04-05 Wed

- [x] API: authentication and authorization

    - HTTPS for security whose request data is encrypted.

    - authentication: priving identity to access all resource
        - login to email accout
        - two factor authentication verify identity twice
    - authorization: grant limited access
        - use token only
    - both authentication and authorization
        - OAuth

- [x] API: OAuth - Open Authorization
    - allow 3rd party access
    - limited access
    - through HTTP

## 2023-04-05 Thu

- [x] API: Postman
    - basic authentication: username + password
    - digest authentication: rarely used, using digest to authenticate
    - bearer token: no authentication needed so anyone can use it. keep it in safe place.
    - OAuth: access API through third party, for example, sign in through Google account.

## 2023-04-06 Fri

- [x] API: swagger, create and deploy API on AWS

## postman API platform tutorial
https://www.postman.com/postman/workspace/15-days-of-postman-for-testers/collection/1559645-032fb22a-9afb-4c56-b8f0-4042db96a4f3


