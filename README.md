# local-macro

A small utility to create and run directory-specific macros.

## Usage

```zsh
lm <macro-name> <args...>
```

## Details

`lm` looks for and runs executable scripts in the `.local-macro` directory, as well as `.local-marco` directories in parent directories.

## Configuration

Configuration is done via environment variables.
Env var | Description | Default
-----|-------------|--------
LM_DIR_NAME | The name of the directory containing the macro scripts | `.local-macro`

## Example

Assume we have a project with the following directory structure:

```bash
- project/
  - frontend/
    - .local-macro/
      - run
      - ...
  - backend/
    - .local-macro/
      - run
      - ...
  - .local-macro/
    - sync-schemas
    - ...
```

with a node frontend

```bash
$ cat frontend/.local-macro/run
#!/bin/zsh
npm run dev
```

and a python backend

```bash
$ cat backend/.local-macro/run
#!/bin/zsh
python manage.py runserver
```

and a set of shared schemas that need to be synced between them

```bash
$ cat .local-macro/sync-schemas
#!/bin/zsh
cp frontend/schemas/schema.json backend/schemas/schema.json
```

Running the frontend and backend and syncing the schemas are all commands which need to be run often, but are only relevant to this project. `lm` makes it easy to define these macros only where they're relevant.

```bash
$ cd project/frontend
$ lm run # calls project/frontend/.local-macro/run
```

```bash
$ cd project/backend
$ lm run # calls project/backend/.local-macro/run
```

```bash
$ cd project/backend
$ lm sync-schemas # calls project/.local-macro/sync-schemas
```

## Writing macros

Macros are simple scripts that are called with the arguments passed to `lm`. They can be written in any language, as long as they are executable. `lm` will pass along all arguments after the macro name to the script.

Additionally, `lm` will pass several environment variables to the script:

| Env var      | Description                                                              |
| ------------ | ------------------------------------------------------------------------ |
| LM_CALL_DIR  | The path to the directory from which `lm` was called                     |
| LM_MACRO_DIR | The path to the parent of the resolved script's `.local-macro` directory |
| LM_MACRO     | The name of the macro                                                    |

These can be used to customize the behavior of the macro.
