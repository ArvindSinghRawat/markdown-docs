# Worflow Configuration

Once, markdown docs are running and building successfully in local, we can then write Github actions to prepare the automatic pipeline to automate this process.

## Workflow definition

Find the definition of automated workflow in file `.github/workflows/build-documentation-site.yml`. Here, each section of that file is described:

1. When this workflow will be triggered:
   ```yml
   on:
   push:
     branches: ["main"]
     paths:
       - "docs/**"
   pull_request:
     branches: ["main"]
     paths:
       - "docs/**"
   workflow_dispatch:
   ```
   This defines that workflow will be triggered when any push is made on the main branch or a pull_request is made on the same, additionally, it also checks if `docs` directory is changed or not. This also supports the `worflow_dispatch` which enables us to run this workflow on the click of a button.

2. Add permissions to enable deployment to Github pages:
   ```yml
   permissions:
    id-token: write
    pages: write
   ```
   This defines that Github token needs additional permissions over the default ones to allow write on github pages.

3. Job Structure:
   ```yml
   jobs:
    build-documentation-site:
        runs-on: ubuntu-latest
        steps:
   ```
   This defines the actual job which is going to run with its name on the top and then, where it is going to run. After that we have an array of steps which are going to be described in subsequent pointers.

4. Checkout code:
   ```yml
      - name: 'Checkout Code'
        uses: actions/checkout@v4
   ```
   As the name suggests, this step checks out our code in the runner.

5. Install and setup python:
   ```yml   
      - name: 'Setup Python'
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip' # caching pip dependencies
    ```
    This installs and setup python in the runner to execute our build script. It also enables a cache on `pip` to cache dependecies for subsequent runs.

6. Install python dependencies
    ```yml
      - name: 'Install dependencies'
        run: pip install -r docs/requirements.txt
    ```
    This step installs all the python dependencies required for our `documentation` site.

7. Build site
   ```yml   
      - name: 'Build Site'
        run: mkdocs build -f docs/mkdocs.yml
   ```
   This step builds our site and generate distribution files out of it.

8. Fix permissions
   ```yml
        - name: 'Fix permissions'
                run: |
                chmod -c -R +rX "docs/site/" | while read line; do
                    echo "::warning title=Invalid file permissions automatically fixed::$line"
                done
   ```
   This step is optional, it fixes the permission for each file (if broken), as the next steps needs the files with very specific permissions.

9. Archive `site` folder
    ```yml
      - name: 'Archive artifact'
        run: |
          tar \
            --dereference --hard-dereference \
            --directory "docs/site/" \
            -cvf "$RUNNER_TEMP/artifact.tar" \
            --exclude=.git \
            --exclude=.github \
            .
        env:
          INPUT_PATH: docs/site/
    ```
    This step creates an archive of our distribution folder which will be later used to upload and deployment to github pages.

10. Upload to Github pages
    ```yml
      - name: 'Upload artifact'
        uses: actions/upload-artifact@v3
        with:
          name: github-pages
          path: ${{ runner.temp }}/artifact.tar
          retention-days: 1
          if-no-files-found: error
    ```
    This uploads the artifact to github to make it accessible for deployment.

11. Deploy to Github pages:
    ```yml
      - name: 'Deploy to GitHub Pages'
        id: deployment
        uses: actions/deploy-pages@v3
    ```
    This step actually uses the uploaded artifact and then deploy it to the Github pages. Once, this step is done our site is ready to be used in the URL provided by the Github pages.