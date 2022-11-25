# How to contribute

Welcome to contribute to OpenCloudOS!

## Contribute to OpenCloudOS

## Contribute to the documentation

1. Fork [the documentation repository](https://github.com/OpenCloudOS/opencloudos.github.io/fork)
2. Clone the forked documentation repository to your local machine

    ```sh
    git clone git@github.com:yourname/opencloudos.github.io.git # (1)
    ```

    1. (You need to replace `yourname` with your own GitHub username)

3. Configure the environment
    - Install Python 3.x
    - Install [mkdocs-material](https://squidfunk.github.io/mkdocs-material/) and multi langual plugin

    ``` sh
    pip install mkdocs-material mkdocs-static-i18n
    ```

    - Run the preview server locally

    ``` sh
    mkdocs serve
    ```

4. Start contributing!
    - Refer to mkdocs-material's [documentation](https://squidfunk.github.io/mkdocs-material/reference/) for more details about markdown and the features supported by this documentation site.
    - Please follow the [format guide](docs-format-guide.en.md) of this documentation site.

5. Verify if the content and format are correct through the preview server, and then commit your changes.
6. Submit a Pull Request to [the documentation repository](https://github.com/OpenCloudOS/opencloudos.github.io), and it will be merged after being reviewed.
