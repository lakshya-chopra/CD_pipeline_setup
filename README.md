# CD_pipeline_setup
- Requirements:
  ```
    Python 3 
    Flask : pip3 install Flask
    Docker : can be read at the link given below
  ```
- To set up a CD pipeline, first we'll create a basic web app written in Python's Flask, for this execute the following commands:
  ```
  $ mkdir pythonWebApp
  $ cd pythonWebApp
  $ mkdir app
  $ touch app/app.py
  $ vi app/app.py
  ```
  
  ```
  In `app.py`, paste the following code:
   
  from flask import Flask
    
    app = Flask(__name__)
    
    
    @app.route('/hello')
    def hello_world():
        return "Hello World! How are you doing?"
    
    
    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=8008, debug=True)
 
    ```
  Save the file and then create a Dockerfile in the `pythonWebApp` directory (not in the app sub-folder):
  ```
  FROM python:3

  #creates a new directory myApp/app in the container
  WORKDIR myApp/app
  RUN apt update
  RUN pip3 install flask
  
  COPY app .
  
  EXPOSE 8008
  
  CMD ["python", "app.py", "runserver", "0.0.0.0:8008"]

  ```
- Run, `docker build -t myapp .` in the parent folder (i.e. `pythonWebApp`)
- After successful completion of the build, run `docker images` to view the image `myapp`.
- Run a container using this image : `docker run -p 8008:8008 myapp`, use -p flag and specify port 8080, so that we can access the container on the 8080 port of our host.
- Open your browser, and enter the localhost url along with the port 8080, image below:
- ![image](https://github.com/lakshya-chopra/CD_pipeline_setup/assets/77010972/c0ca9d9b-f4cb-4d7a-a2ef-4cdb4b22399e)
- Congrats! your first containerized application is successful.

## Setting up a CD pipeline using Jenkins:
- Install Java (preferably OpenJDK-17):
  `sudo apt-get install openjdk-17-jdk`, then
  test it using `java -version`.
- Later you may add Java to global environment variables so that other applications can access it.
- Install git
- Install Jenkins:
    ```
    $ wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key |sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg
    $ sudo sh -c 'echo deb [signed-by=/usr/share/keyrings/jenkins.gpg] http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    $ sudo apt install jenkins
    $ #enable and start
    $ sudo systemctl enable jenkins
    $ sudo systemctl start jenkins
    $ sudo systemctl status jenkins
    ```
- Allow Jenkins default port 8080:
  ```
  $ sudo ufw allow 8080
  $ sudo ufw status
  $ sudo ufw enable
  $ sudo ufw status
  ```
- After this, login in into Jenkins by going to it’s default link: http://127.0.0.1:8008, here enter the initialAdminPassword.

## GitHub Webhook setup:
-  In your Jenkins dashboard, go to `Manage Jenkins -> System Configuration` and from there set up the Github server with your credentials:
- ![image](https://github.com/lakshya-chopra/CD_pipeline_setup/assets/77010972/a97e68dd-868d-441d-8589-094bae0278b5)

- Navigate the settings of your github project, and create a webhook which will send a notification to your app whenever you do some changes to your github repo:
- ![image](https://github.com/lakshya-chopra/CD_pipeline_setup/assets/77010972/84edf10b-6fbd-4d03-9be0-adb8542ec9e6)

- ![image](https://github.com/lakshya-chopra/CD_pipeline_setup/assets/77010972/2ffc188c-3687-41b5-8f82-e8adf8c5bb49)
- ![image](https://github.com/lakshya-chopra/CD_pipeline_setup/assets/77010972/8c0491ae-cab0-49e3-a05a-4e161ab0b5b9)
- ![image](https://github.com/lakshya-chopra/CD_pipeline_setup/assets/77010972/e571636f-92f4-478c-9fbb-89dd3d72caca)
- ```
  pipeline {
    agent any

    stages {
        stage("Clone Code") {
            steps {
                echo "Cloning the code"
                git url: "https://github.com/lakshya-chopra/pythonWebApp.git", branch: "main"
            }
        }

        stage("Build") {
            steps {
                echo "Building the Docker image"
                sh "docker build -t myapp ."
            }
        }


        stage("Deploy") {
            steps {
                echo "Deploying the container"
                sh "docker run -it -p 8080:8080 myapp"
                
            }
        }
    }
  }
```




