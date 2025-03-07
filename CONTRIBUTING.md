# Contributing

This repository uses [mdBook](https://github.com/rust-lang/mdBook) to render markdown files stored in `src`.

## Getting started

1. Install [Rust](https://www.rust-lang.org) with

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```

2. Install [mdBook](https://github.com/rust-lang/mdBook) with

    ```rust
    cargo install mdbook mdbook-linkcheck
    ```

3. Run `mdbook serve --open` to build to build the site and open it locally.

## Things to note

- Pages in folders are not automatically added to the site. They must be linked to in <src/SUMMARY.md>.
- Images can technically be linked to from anywhere, but best practice is the place them in the folder of the file using them, and to give files that embed a lot of images their own folder.
- `mdBook-linkcheck` errors on broken internal links.
