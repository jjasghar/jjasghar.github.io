---
layout: post
title: "Creating a local copilot using vetted LLMs"
date: 2024-06-17 17:21:23
categories: ai linux programming
---

First off, I want to thank [Daniel Avila][daniel] for his post [here][main].
It inspired [Paul Czarkowski][paul] and I to create this [video][video] on how to build
an IBM and Red Hat based version. Thank you Daniel!

## Installing `ollama`

In order to jump down this path, you need to have a way to run local LLMs on your laptop. Luckily that's where
`ollama` comes into play. You can do your research [here][ollama] but the short of it, it's a an application
that will host/run the LLM on you laptop.

1. Head on over to <https://ollama.com/download> and download your version of `ollama` and start it up.
2. Next you need to pull down (in this case) trusted LLM of `granite-code`.
   ```bash
   ollama pull granite-code:8b
   ```
   **Note**: as of writing this article you can run the following versions, I would start with `8b` and change if you
   need to.
   - 34B Parameters `ollama pull granite-code:34b`
   - 20B Parameters `ollama pull granite-code:20b`
   - 8B Parameters `ollama pull granite-code:8b`
   - 3B Parameters `ollama pull granite-code:3b`
3. Run your model to verify everything is good: `ollama run granite-code`, ask it some questions see that it's
answering how you'd like.
4. If you want you can create a `Modelfile` like the following example if you want to play with any of options:
   ```dockerfile
   FROM granite-code

   # sets the temperature to 1 [higher is more creative, lower is more coherent]
   PARAMETER temperature 1

   # sets the context window size to 1500, this controls how many tokens the LLM can use as context to generate the next token
   PARAMETER num_ctx 1500

   # sets a custom system message to specify the behavior of the chat assistant
   SYSTEM You are expert Code Assistant, who wants to give the easiest read code possible.
   ```
5. If you want to change the things around with the `Modelfile` create the new version of the model:
   ```bash
   ollama create autopilot -f ./Modelfile
   ```
6. Run `ollama list` to verify you find it.
7. Sanity check model via another `ollama run autopilot` to make sure it's working how you want.

## VSCode

Now that you have a local `autopilot` running lets get VSCode working with it. You can technically host
it in a shared location, but for right now we are going to run this locally.

1. Go to <https://www.continue.dev> and download the version for VSCode. (you can use JetBrains if you want but it's
out of scope of this post).
2. After installing it into VSCode you should be able to see, "Continue" in the corner.
![](https://ollama.com/public/blog/continue-settings-vscode.png)
3. Right click it and bring up your `config.json`, and edit it this way:
```json
{
  "models": [
    {
      "title": "autopilot",
      "provider": "ollama",
      "model": "autopilot"
    }
  ],
  "tabAutocompleteModel": {
    "title": "autopilot",
    "provider": "ollama",
    "model": "autpilot"
  }
}
```

With that, you should be able to start playing with Continue and your local `autopilot` model!

[video]: https://www.youtube.com/watch?v=7ItGnj5ORYk
[ollama]: https://ollama.com/
[daniel]: https://medium.com/@dan.avila7
[main]: https://blog.codegpt.co/create-your-own-and-custom-copilot-in-vscode-with-ollama-and-codegpt-736277a60298
[paul]: https://github.com/paulczar
