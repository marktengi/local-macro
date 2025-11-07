# local-macro

A small utility for creating and running directory-local command macros.

`lm` lets you define simple (or complex, if you want) commands that can be stored anywhere in your directory tree. When `lm macro-name [args...]` is run, `lm` will recursively search up the tree for a macro named `macro-name` and run it, passing along any arguments you provide. This allows you to define hierarchical macros that can be overridden by more specific macros in subdirectories.

## Installation

The easiest way to install is to just clone the repository and symlink the `lm` executable to a directory on your `PATH`.

## Quick Start

1.  **Create your first macro:**

    `lm` makes it easy to create a macro for your current directory:

    ```bash
    lm -c my-cool-macro
    ```

    This will:

    - Create a new directory called `.local-macro` in the current directory if it doesn't already exist.
    - Create a new file called `my-cool-macro` in `./.local-macro/`.
    - Make the file executable.
    - Open the file in your editor.

2.  **Make it do something:**

    ```bash
    $ echo 'echo "Hello, from local-macro!"' > .local-macro/my-cool-macro
    ```

3.  **Run the macro:**
    ```bash
    $ lm my-cool-macro
    >> Hello, from local-macro!
    ```

## How It Works

When you run `lm my-cool-macro`, `lm` searches for an executable file named `my-cool-macro` in the following locations, in order:

1.  `./.local-macro/my-cool-macro` in the current directory.
2.  `../.local-macro/my-cool-macro` in the parent directory.
3.  `../../.local-macro/my-cool-macro` in the grandparent directory.
4.  ...and so on, up to the root directory.

The first executable macro it finds is the one it runs. If it doesn't find one, it will print an error message and exit with a non-zero status code.

## Usage

```
lm macro-name [args...]
lm -c | --create macro-name
lm -l | --list
lm -r | --resolve macro-name
lm -h | --help
```

### Commands

| Command                            | Description                                                                             |
| ---------------------------------- | --------------------------------------------------------------------------------------- |
| `lm macro-name [args...]`          | Runs the macro with the given name and arguments.                                       |
| `lm`                               | Lists all macros in the current directory and parent directories.                       |
| `lm -c name` or `lm --create name` | Creates a new macro with the given name in the current directory.                       |
| `lm -l` or `lm --list`             | Lists all macros in the current directory and parent directories.                       |
| `lm -r` or `lm --resolve <name>`   | Recursively resolves the given macro name to the script path and prints it to `stdout`. |
| `lm -h` or `lm --help`             | Shows the help message.                                                                 |

## Example

Consider a project with a frontend, a backend, and a shared schema:

```
- project/
  - .local-macro/
    - sync-schemas
  - frontend/
    - .local-macro/
      - run
  - backend/
    - .local-macro/
      - run
```

The `run` macro can be defined differently for the frontend and backend:

**`frontend/.local-macro/run`**:

```bash
#!/bin/sh
npm run dev
```

**`backend/.local-macro/run`**:

```bash
#!/bin/sh
python manage.py runserver
```

And a shared macro to sync schemas:

**`.local-macro/sync-schemas`**:

```bash
#!/bin/sh
cp frontend/schemas/schema.json backend/schemas/schema.json
```

Now you can run your project-specific commands easily:

```bash
# Start the frontend server
$ cd project/frontend
$ lm run

# Start the backend server
$ cd project/backend
$ lm run

# Sync schemas from anywhere in the project
$ cd project/frontend
$ lm sync-schemas
```

## Writing Macros

Macros are executable scripts that can be written in any language. `lm` passes any arguments after the macro name directly to your script.

`lm` also sets the following environment variables when it runs a macro:

| Env Var        | Description                                      |
| -------------- | ------------------------------------------------ |
| `LM_CALL_DIR`  | The directory from which `lm` was called.        |
| `LM_MACRO_DIR` | The directory where the macro script is located. |
| `LM_MACRO`     | The name of the macro being executed.            |

These variables allow you to write more powerful and context-aware macros.

## Configuration

You can customize the name of the macro directory via an environment variable:

| Env Var       | Description                                  | Default        |
| ------------- | -------------------------------------------- | -------------- |
| `LM_DIR_NAME` | The name of the directory containing macros. | `.local-macro` |

## Inspiration

Inspired by [a post from Evan Hahn][1] and the corresponding [Hacker News discussion][2].

[1]: https://evanhahn.com/scripts-i-wrote-that-i-use-all-the-time/
[2]: https://news.ycombinator.com/item?id=45670052
