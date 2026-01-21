# Instructions for the Scaffolded Node Application

The idea is to build this app incrementaly.

# Phase 0

- Remember to setup git and github to be able to push and pull

The steps to follow are:
1. Create a repository in your GitHub account, call it `scaffolded-books`
    1. Remember to add a README file
    1. Remember to add a .gitignore for Node
2.	Clone this repository into your computer (Remember where you cloned it)
3.	Move to the directory where you cloned your repository (most likely `/development/scaffolded-books`)
4.	Run the following command

    `mkdir -p .devcontainer`
5. Create the following two files:
    1. `.devcontainer/Dockerfile`
    2. `.devcontainer/devcontainer.json`
6. Write the following code on the `Dockerfile` file
```dockerfile
# .devcontainer/Dockerfile
FROM node:20-bookworm

# Optional: install a few useful tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
  && rm -rf /var/lib/apt/lists/*

# Workspace folder (VS Code will mount your repo here)
WORKDIR /workspaces

# Ensure Node/npm versions visible
RUN node -v && npm -v
```
7. Write the following code on the `devcontainer.json` file
```json
{
  "name": "Node Scaffold (Phase-0)",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",

  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  },

  "forwardPorts": [3000],
  "portsAttributes": {
    "3000": {
      "label": "Node app",
      "onAutoForward": "notify"
    }
  },

  "postCreateCommand": "node -v && npm -v"
}
```
8. Verify that the `.gitignore` file correctly ignores Node related binaries.
9. Add to the `.gitignore` the following lines at the end:
```
# OS / Editor
.DS_Store
.vscode/
```
10. Create a file `./.editorconfig` (This file is in your repo's root). Write the following code into it:
```ini
root = true

[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8

[*.{js,json,md}]
indent_style = space
indent_size = 2
```

11. Now, in the terminal, while in the repo's root directory type: `code .` this will open Visual Studio Code in the current directory. This has the same effect as opening Visual Studio Code (VSC) and then opening that folder.
12. VSC will tell you "Folder contains a Dev Container configuration..." Click on the button that says "Reopen in Container." This will make VSC read the configuration files we just wrote before, and interact with Docker to create a container for our application. From now on, you will work on that container and not "on your computer"
13. Within VSC create a file `smoke.js` and type the following code:
```js
// smoke.js

const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Phase-0 smoke test OK\n");
});

server.listen(3000, "0.0.0.0", () => {
  console.log("Listening on http://0.0.0.0:3000");
});

```
14. Run your mini-program, open a Terminal from within VSC and run:
```bash
node smoke.js
```
15. You can open a web browser on your computer and type this url: `http://localhost:3000` You should get a message that reads "Phase-0 smoke test OK"
16. You now can commit and push, run the following commands:
```bash
git add .
git commit -m "Phase-0: devcontainer + dockerized Node environment"
git tag phase-0
git tag
git push --tags

```



