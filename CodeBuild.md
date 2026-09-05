What is <b> CodeBuild </b> ?
AWS CodeBuild is a fully managed build service that compiles source code, runs tests, and produces software packages that are ready to deploy. With CodeBuild, you don’t need to worry about provisioning and managing your own build infrastructure. You simply provide your build project’s source code and build settings, and CodeBuild handles the rest.

For example, if you have a web application that you want to deploy, you can use CodeBuild to compile your source code, run unit tests, and produce a deployable package. You can also use CodeBuild to build Docker images, run static code analysis, and more. CodeBuild integrates with other AWS services like CodePipeline, so you can easily automate your entire software release process.

Here is our Today’s Task-01 :


- What is Buildspec file for Codebuild.
The Buildspec file is a configuration file used by AWS CodeBuild to define how to build and deploy your application or software project. It is written in YAML or JSON format and contains a series of build commands, environment variables, settings, and artifacts that CodeBuild will use during the build process.



<img width="1366" height="661" alt="image" src="https://github.com/user-attachments/assets/ac717dc0-21e4-4e45-b981-bb08d93d0a99" />


<img width="1366" height="661" alt="image" src="https://github.com/user-attachments/assets/10b39bcb-13b7-4dba-995a-dcc630a71630" />


<img width="1366" height="661" alt="image" src="https://github.com/user-attachments/assets/4b925ad6-d919-49d4-8ca6-2767ea591b33" />


Enter “index.html” as the file name and add the following content in the editor:
<html>
  <head>
    <title>My Website</title>
  </head>
  <body>
    <h1>Welcome to my website</h1>
    <p>This is a simple website hosted on AWS CodeCommit.</p>
  </body>
</html>

<img width="1366" height="1238" alt="image" src="https://github.com/user-attachments/assets/e971cc4a-4e3b-4c9b-954d-fbed75d25ee7" />


<img width="1366" height="1238" alt="image" src="https://github.com/user-attachments/assets/059c6181-980c-4166-bd53-4ca31c161fb9" />


<img width="1366" height="1238" alt="image" src="https://github.com/user-attachments/assets/58a3db7a-747a-4505-a9ae-8e6c1b6aa478" />


<img width="1366" height="661" alt="image" src="https://github.com/user-attachments/assets/3c3b07f4-3af6-4221-847d-5787d149a0d5" />


<img width="1366" height="661" alt="image" src="https://github.com/user-attachments/assets/81d0a7c5-d4e8-427e-afb5-be30d8af9a9b" />


<img width="1366" height="661" alt="image" src="https://github.com/user-attachments/assets/bc7b4e4c-e7b2-411c-a805-29992876afeb" />



# Adding a buildpsec.yml file
Create a buildspec.yaml file that will contain the pre-build,during build,post-build, storage artifacts to be performed on the server during the project build.


