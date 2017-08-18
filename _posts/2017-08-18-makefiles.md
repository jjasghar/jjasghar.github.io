---
layout: post
title: "Makefiles, yep"
date: 2017-08-18 12:00:46
categories: linux sysadmin
---

I came across this [post][afraid] and relearned about `Makefiles`. I had completely
forgotten how awesome they are, and that I should be using them more. I've created
a template hear inspired from the post that if anything might help me in the future.

```make
# This is the default action, it's always the first target
# .PHONY <target> at the end of the build step. Common phony targets
# are: clean, install, run,...
# Otherwise, if somebody creates an install directory, make will
# silently fail, because the build target already exists.
.PHONY: help print

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

# make target: <dependancies> ## comment for the self documenting
dress: trousers shoes jacket ## The main goal of this Makefile
	@echo "All done. Let's go outside!"

# make target: <dependancies> ## comment for the self documenting
jacket: pullover ## Gotta put on the jacket
	@echo "Putting on jacket."

# make target: <dependancies> ## comment for the self documenting
pullover: shirt ## Putting on the pullover
	@echo "Putting on pullover."

# make target: <dependancies> ## comment for the self documenting
shirt: ## A shirt is the begining of top half of the body
	@echo "Putting on shirt."

# make target: <dependancies> ## comment for the self documenting
trousers: underpants ## Trousers are important for outside
	@echo "Putting on trousers."

# make target: <dependancies> ## comment for the self documenting
underpants: ## You need underwear on
	@echo "Putting on underpants."

# make target: <dependancies> ## comment for the self documenting
shoes: socks ## Shoes are important outside
	@echo "Putting on shoes."

# make target: <dependancies> ## comment for the self documenting
socks: pullover  ## Blisters suck, these help stop those
	@echo "Putting on socks."

# make target: <dependancies> ## comment for the self documenting
print: ## Output the Makefile, demo more then just @echo
	@cat Makefile
```

[afraid]: https://matthias-endler.de/2017/makefiles/
