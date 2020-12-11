id: ci-cd-workshop-source

summary: This is the workshop of the CI/CD implementation, on the Android Academy series.

authors: Artyom Okun, Nitzan Werber

# Android Academy CI/CD Workshop Dec 2020
<!-- ------------------------ -->
## Repository Setup
Duration: 0:05:00

1. Create a new application in Android Studio (or use your own previously created one).
2. Build and compile a project.
3. Create a Github repository for this project(or skip to the step 4 if you already have one). You can follow the instructions [here](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/create-a-repo).
4. Optional step - Create first commit (we just want to verify your local copy is synced with Github repo and we can start work on CI/CD). From the app root folder, open terminal and run:

```bash
	git add .
	git commit -m "Initial commit"
	git push origin master
``` 

#### Congrats! We can start with CI related files now :)

<!-- ------------------------ -->
##  Run your first CI build
Duration: 0:05:00

### Add workflow file

<span>1.</span> Go to project and a directory `.github` in the root.<br/>
<span>2.</span> Inside it, create another directory called `workflows` (This is where all the GitHub Actions configuration files go).<br/>
<span>3.</span> Create first configuration file `workflow_1.yaml`.<br/>
<span>4.</span> Open it and add next code (make sure the indentation is exact as in the example, as yaml is sensitive to indentations):

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
git commit -m "Initial CI script"
git push origin master
```
<br/>
<span>6.</span> You should see a result:
![image_caption](resources/initial-build-result.png)

<!-- ------------------------ -->

## Add Unit Tests
Duration: 0:03:00

### First run unit tests locally

<span>1.</span> Go to your test folder.<br/>
<span>2.</span> Locate `ExampleUnitTest.kt` or other test files and validate the test passes locally by running in command line<br/>

``` bash
./gradlew testDebugUnitTest
```

### Modify `workflow_1.yaml` file:

<span>1.</span> Add to the bottom of your workfow file:<br/>

```yaml
- name: Unit tests
  run: ./gradlew testDebugUnitTest
```

<span>2.</span> Commit and push the changes:

``` bash
git commit -m "Add unit test to the workflow"
git push origin master
```
<span>3.</span> Verify your remote CI build triggers and passes.


<!-- ------------------------ -->

##  Convert to (unsigned) release build
Duration: 0:03:00

### Modify `workflow_1.yaml` file

<span>1.</span> Change from:<br/>`./gradlew assembleDebug` to `./gradlew assembleRelease`<br/>
<span>2.</span> Change from:<br/> `./gradlew testDebugUnitTest` to `./gradlew testReleaseUnitTest`<br/>
<span>3.</span> Commit ang push the changes:

``` bash
git commit -m "Convert build to release"
git push origin master
```
<br/>

#### Now we have a remote build, that creates release APK, but it is unsigned 😢. 

<!-- ------------------------ -->

## Signing your release

### Create release.keystore file if not exist yet (skip this step if already have one)

1. Go to ``Android Studio -> Build -> Generate Signed Bundle or APK``.
2. Choose APK and click ``Next``.
3. In the `Keystore Path` click `Create new...`.
4. Follow wizard instructions, fill relevant data and remember created keystore location.
5. Continue the wizard and build release.apk, just to verify that we are able to build a release version of the app locally.

### Modify your app/build.gradle

Add next lines inside `android` closure:

```gradle
    signingConfigs {

        //... 

        release {
            storeFile file('keystore.release')
            keyAlias System.getenv("MY_APP_KEY_ALIAS")
            storePassword System.getenv("MY_APP_STORE_PASSWORD")
            keyPassword System.getenv("MY_APP_KEY_PASSWORD")
        }

        buildTypes {
            release {
                // ...
                signingConfig signingConfigs.release
                // ...
            }
        }
    }

```

### Configure environment variables in the Github Repository

<span>1.</span> Go to the github project repository.<br/>
<span>2.</span> Follow the Settings tab.<br/>
<span>3.</span> In the Settings tab go to Secrets on the left menu (If you can't see it - maybe you don't have permissions for this project).<br/>

![image_caption](resources/Settings_Secrets_tab_github.png)

<span>4.</span>  Now click on the "New Repository Secret" on the right top.<br/>
<span>5.</span>  Give it a name `MY_APP_KEY_ALIAS` and a value you chosen before `your value` and click `Add secret`:<br/>
![image_caption](resources/secrets_added.jpg)<br/>
<span>6.</span>  Repeat steps 4-5 for the `MY_APP_STORE_PASSWORD` and `MY_APP_KEY_PASSWORD`.<br/>

### Upload release.keystore to Github secrets

As we can not upload any files to Github secrets besides strings, we are going to convert our release.keystore to base64 string, store it and during the build process we will convert it back to file.

#### Generate base64 string from release.keystore file

1. Open terminal in the folder where the keystore located at.
2. Run `base64 -w 0 release.keystore | xclip -selection clipboard`(It will also copy the created string to your clipboard).

#### Add keystore string to Github secrets (same as we did for the passwords and alias)

1. Go to your `Github repository -> Settings -> Secrets`.
2. Create new secret and give it name `ENCODED_KEYSTORE`.
3. Paste previously copied string as secret value and save the secret.

#### Modify `workflow_1.yaml` file

<span>1.</span> Add next lines after `Grant rights`:<br/>

``` yaml
- name: Restore release keystore
  run: echo "${{ secrets.SIGNING_KEY }}" | base64 --decode > keystore.release
```
<span>2.</span> Also modify current `Generate APK` step:

``` yaml
- name: Generate APK
  run: ./gradlew assembleRelease
  env:
    MY_APP_STORE_PASSWORD: ${{ secrets.MY_APP_STORE_PASSWORD }}
    MY_APP_KEY_PASSWORD: ${{ secrets.MY_APP_KEY_PASSWORD }}
    MY_APP_KEY_ALIAS: ${{ secrets.MY_APP_KEY_ALIAS }}
```
<br/>
<span>3.</span> Commit and push your code and verify build passes.

#### Now we have a signed release.apk that we can distribute to our testers. Let's see how to do it ih the next step.