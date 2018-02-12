---
layout: post
title: "The steps to create, upload, and run a custom InSpec profile via Chef Automate"
date: 2018-02-12 15:03:24
categories: chef inspec
---

Here are the steps to create, upload, and run  a custom inspec profile via [Chef Automate](https://www.chef.io/automate/).

1. Be sure you have the [ChefDK][chefdk] installed.
2. Create a skeleton profile: `inspec init profile <name>`
3. Edit the `<name>/inspec.yml` with everything you might need, including any [dependencies](https://github.com/chef/inspec-aws#adapt-the-inspecyml).
4. Add a control to `<name>/controls/example.rb`. (You probably want to change this file name.)
5. Verify the controls after you are done: `inspec check <name>`.
6. (_Optional_) Run the profile locally: `inspec exec <name>`.
7. Upload the profile to Automate: `inspec compliance upload <name>`.
8. Verify the profile is uploaded correctly: `inspec compliance profiles`.
9. Run the profile via Automate: `inspec compliance exec YOURUSERNAME/<name>`.

[chefdk]: https://downloads.chef.io/chef-dk/
