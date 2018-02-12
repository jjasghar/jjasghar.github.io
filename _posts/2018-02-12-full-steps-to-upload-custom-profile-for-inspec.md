---
layout: post
title: "The steps to create, upload, and run a custom InSpec profile via Chef Automate"
date: 2018-02-12 15:03:24
categories: chef inspec
---

Here are the steps to create, upload, and run a custom [InSpec](https://www.inspec.io/) profile via [Chef Automate](https://www.chef.io/automate/).

1. (_Optional_) Have the [ChefDK][chefdk] installed, or InSpec installed.
2. Create a skeleton profile: `inspec init profile <name>`
3. Edit the `<name>/inspec.yml` with everything you might need, including any [dependencies](https://github.com/chef/inspec-aws#adapt-the-inspecyml).
4. Add a control to `<name>/controls/example.rb`. (You probably want to change this file name.)
5. Verify the controls after you are done: `inspec check <name>`.
6. (_Optional_) Run the profile locally: `inspec exec <name>`.
7. Login to the Chef Automate instance via InSpec: `inspec compliance login`.
8. Upload the profile to Automate: `inspec compliance upload <name>`.
9. Verify the profile is uploaded correctly: `inspec compliance profiles`.
10. Run the profile via Automate: `inspec compliance exec YOURUSERNAME/<name>`.

[chefdk]: https://downloads.chef.io/chef-dk/
