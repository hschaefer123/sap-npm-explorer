# sap-npm-explorer
Discover and consume digital content packages with APIs from SAP

Features:
## Requirements
- [Python](https://www.python.org/) MkDocs supports Python versions 3.5, 3.6, 3.7, 3.8, and pypy3.
    ```python --version```

- pip (should come with pyhton)
    ```pip --version```
    ```pip install --upgrade pip```

### install MKDocs and dependencies
```
pip install mkdocs
pip install mkdocs-material
pip install codehilite
pip install minify
pip install awesome-pages
```

## Usage
1. Preview documentation

   Start the integrated mkdocs server

    ```mkdocs serve```
   
    and preview generated docs inside browser using http://127.0.0.1:8000

2. Build documentation to /site

    Build the previewed documentation for deployment into "/site" directory.

    ```mkdocs build```

3. Deploy to gh_pages

    ```mkdocs gh-deploy```

6. Once the application has been pushed successfully, open the URl

    https://udina-docs.cfapps.eu10.hana.ondemand.com/

    in a web browser. 