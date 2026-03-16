I used to think whether it is possible to host my own obsidian vault online. In just a few years, that went from "probably possible?" to "most probably possible!".

---
## Step 1: Editor setup
Install [Obsidian](https://obsidian.md/) in your computer. Obsidian should act as your editor and viewer, and it should just be that. For most people, this is probably sufficient, and you wouldn't need to continue further than this.

## Step 2: Repo setup
Follow this tutorial from [Quartz docs](https://quartz.jzhao.xyz/). You should have a `v4` branch by default. As Quartz uses node, its best to install some node management tools. I usually use `nvm` to manage my node environments. 

Install `nvm` and do `nvm use v22.17.0`. 

Then run `npm i`. 

Then `npx quartz create`. Go through the setup process.

## Step 3: Preview
You can (and should!) run `npx quartz build --serve` to view how the page would look like in browser.

