id: ci-cd-workshop-source

summary: This is the workshop of the CI/CD implementation, on the Android Academy series.

authors: Artyom Okun, Nitzan Werber

# Android Academy CI/CD Workshop Dec 2020
<!-- ------------------------ -->
##  Step 1 - Clone or Checkout Our Starter Project
Duration: 0:03:00

### If you've attended Android Academy's previous workshops

- Make sure your starter project branch is up to date.

### If you just came here - welcome! Please clone the the project repository 

- In the following link: <https://github.com/AcademyTLV/StartFromScratch>
- Go to branch "ci_cd_start" 

<!-- ------------------------ -->
## Step 2 - Configure environment variables
Duration: 0:03:00

An environment variable is a dynamic-named value that can affect the way running processes will behave on a computer.

### When running on your local machine there are few ways to use it: 

- run in terminal:

 ```bash 
export MY_ENVIROMENT_VARIABLE = some_value
 ```
 
- add to your `bashrc` file - your bash config file: 

 ``` bash
export MY_ENVIROMENT_VARIABLE = some_value 
```

### But when we run a process the Github Actions' remote machine we define the environment variable in another way:
<span>1.</span> Go to the github project repository.<br/>
<span>2.</span> Follow the Settings tab.<br/>
<span>3.</span> In the Settings tab go to Secrets on the left menu (If you can't see it - maybe you don't have permissions for this project).<br/>

![image_caption](resources/Settings_Secrets_tab_github.png)

<span>4.</span>  Now click on the "New Repository Secret" on the right top.<br/>
<span>5.</span>  Add a new secret!<br/>
![image_caption](resources/New_secret.png)


// TODO Nitzan - continue from here
<!-- ------------------------ -->
##  Step 3 - Run your first build
Duration: 0:05:00

### Add workflow file

<span>1.</span> Go to project and a directory `.github` in the root.<br/>
<span>2.</span> Inside it, create another directory called `workflows` (This is where all the GitHub Actions configuration files go).<br/>
<span>3.</span> Create first configuration file `workflow_1.yaml`.<br/>
<span>4.</span> Open it and add next code:

```yaml
name: "CI Workflow"
on: [push]
	
jobs: 
  build: 
    name: "Build project"
    runs-on: ubuntu-latest
    steps: 
      - name: "Checkout current repository in ubuntu's file system"
        uses: actions/checkout@v1
      - name: "Setup JDK 1.8"
        uses: actions/setup-java@v1
        with: 
          java-version: 1.8
      - name: "Builds debug build"
        run: ./gradlew assembleDebug
```
<br/>
<span>5.</span> Commit ang push the changes to origin:

``` bash
git add .
git commit -m "Initial build action add"
git push origin master
```
<br/>
<span>6.</span> You should see a result:
![image_caption](resources/initial-build-result.png)

<!-- ------------------------ -->
