
The main objective of the exercise is to add a Dockerfile to a Python App. So we have an application and we are asked to do the following:
* we need to create a Dockerfile that runs our python application and expose on a well defined port, let's say the 9000. On that port the app will return the environment variable `ENVIRONMENT=production` ( unless we will ask to return something different ) 

!!! tip
    You can get the files needed to complete the exercise either from git `git clone  https://github.com/spigad/simple-exercise.git` or just cut&paste the code here below. 

### The App

A simple python3 API that only responds at `/`. It returns the value
of the `ENVIRONMENT` environment var as JSON.

```python
from flask import Flask
import os
import sys

app = Flask(__name__)

@app.route("/")
def index():
    env = os.getenv("ENVIRONMENT", None)
    return {
        "env": env
    }

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Needs port number as a commandline argument.")
        sys.exit(1)
    port = int(sys.argv[1])
    app.run(host='0.0.0.0', port=port)
```

### The  Dependencies

Our App has some dependences, so we need to take care of them, here below the list of the software dependences:

```
Click==7.0
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.11.3
MarkupSafe==1.1.1
Werkzeug==0.16.0
```

!!! tip
    We will Use `pip` to install the requirements listed in `requirements.txt`:
    * `pip install -r requirements.txt`

### Running the server

Of course we want our App be up&running. The server requires one command line argument: the port that it should listen on. 
To run the app we need just something like 

`python app.py <PORT>`

### Compose the Dockerfile

Let's now build the Dockerfile in order to pack our app

```Dockerfile
FROM python:3.7-slim-buster

RUN apt-get update && apt-get install -y curl

COPY requirements.txt /tmp/

RUN pip install -r /tmp/requirements.txt

ENV PORT="3000"

EXPOSE ${PORT}

RUN useradd --create-home pythonappuser
WORKDIR /home/pythonappuser

COPY startup.sh .
COPY app.py .

RUN chmod +x startup.sh && chmod +x app.py && chown -R pythonappuser:pythonappuser .
USER pythonappuser

ENTRYPOINT ["./startup.sh"]
```
