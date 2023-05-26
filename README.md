# SQL-Injection-Scanner

## Usage
This script is intended to locate SQL Injection vulnerabilities within single paged websites that were built in HTML, CSS, and JavaScript (no frameworks). 

## Warning
Please do not use this script on websites you do not have permission to test on or on websites you do not own!

## Explaination
```
import requests
from bs4 import BeautifulSoup
import sys
from urllib.parse import urljoin

s = requests.Session()
s.headers["User-Agent"] = "enter your browser's user agent here"
```
The code imports the necessary libraries: requests, BeautifulSoup, sys, and urljoin from urllib.parse.
It creates a requests.Session() object s to maintain the session and handle multiple requests.
It sets the User-Agent header for the session. You should replace "enter your browser's user agent here" with the User-Agent header string from your browser. The User-Agent header helps to identify the client (browser) making the request.
```
def get_forms(url):
    soup = BeautifulSoup(s.get(url).content, "html.parser")
    return soup.find_all("form")
```
This function get_forms takes a URL as input and returns a list of all the HTML forms present on the webpage.
It uses requests.get to fetch the webpage content and then creates a BeautifulSoup object to parse the HTML content.
BeautifulSoup is initialized with the webpage content and the parser used is "html.parser".
It uses soup.find_all("form") to find all the <form> tags in the HTML and returns them as a list. 
```
def form_details(form):
    detailsOfForm = {}
    action = form.attrs.get("action")
    method = form.attrs.get("method", "get")
    inputs = []

    for input_tag in form.find_all("input"):
        input_type = input_tag.attrs.get("type", "text")
        input_name = input_tag.attrs.get("name")
        input_value = input_tag.attrs.get("value", "")
        inputs.append({
            "type": input_type,
            "name" : input_name,
            "value" : input_value,
        })

    detailsOfForm['action'] = action
    detailsOfForm['method'] = method
    detailsOfForm['inputs'] = inputs
    return detailsOfForm
```
This function form_details takes a form object as input and returns a dictionary containing details about the form.
It extracts the form's action, method, and inputs.
The action attribute represents the URL where the form data should be submitted.
The method attribute specifies the HTTP method to be used ("get" is the default if not specified).
The function iterates over each <input> tag within the form and extracts the input's type, name, and value attributes.
The input details are stored as dictionaries in the inputs list.
Finally, the function returns a dictionary (detailsOfForm) with the action, method, and inputs.
```
def vulnerable(response):
    errors = {"Quoted string not properly terminated",
              "unclosed quotation mark after the character string",
              "you have an error in your SQL syntax"}
    for error in errors:
        if error in response.content.decode().lower():
            return True
    return False
 ```
This function vulnerable takes a response object as input and checks if it contains known SQL injection error messages.
It defines a set of common error messages associated with SQL injection vulnerabilities.
The function iterates over each error message and checks if it exists in the response content (converted to lowercase).
If any of the error messages are found, it returns True, indicating a potential SQL injection vulnerability. Otherwise, it returns False.
```
def sql_injection_scan(url):
    forms = get_forms(url)
    print(f'[+] Detected {len(forms)} forms on {url}.')

    for form in forms:
        details = form_details(form)

        for i in "\"'":
            data = {}
            for input_tag in details["inputs"]:
                if input_tag["type"] == "hidden" or input_tag["value"]:
                    data[input_tag['name']] = input_tag["value"] + i
                elif input_tag["type"] != "submit":
                    data[input_tag['name']] = f"test{i}"

            print(url)
            form_details(form)

            if details["method"] == "post":
                res = s.post(url, data=data)
            elif details['method'] == "get":
                res = s.get(url, params=data)
            if vulnerable(res):
                print("SQL Injection Attack vulnerability in link: ", url)
            else: 
                print("No SQL Injection attack vulnerability detected")
                break
  
  ```
This function sql_injection_scan takes a URL as input and scans for SQL injection vulnerabilities in the forms present on the webpage.
It first calls the get_forms function to retrieve all the forms on the webpage.
It then iterates over each form and extracts its details using the form_details function.
For each form, it creates different payloads by appending " and ' to the input values. It tests both single and double quote characters.
It creates a dictionary data to store the modified form input values.
It then sends requests to the URL with the modified payload and the appropriate HTTP method (post or get).
The response is checked using the vulnerable function to detect potential SQL injection vulnerabilities.
If a vulnerability is detected, it prints a message indicating the vulnerability and the URL.
If no vulnerability is found, it prints a message indicating no SQL injection vulnerability and breaks the loop.
```
if __name__ == "__main__":
    urlToBeChecked = 'https://houseofcode.info'
    sql_injection_scan(urlToBeChecked)
```
This block of code is executed when the script is run directly, not imported as a module.
It defines a variable urlToBeChecked with the URL you want to scan for SQL injection vulnerabilities.
It calls the sql_injection_scan function with the specified URL.
To use this script, you can replace the value of urlToBeChecked with the URL you want to scan. Make sure you have the necessary libraries (requests, BeautifulSoup, etc.) installed before running the script.
