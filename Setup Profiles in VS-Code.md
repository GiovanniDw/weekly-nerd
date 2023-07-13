# Setup Profiles in VS-Code

## Why I created Multiple Profiles for Front-End Development in VS-Code

When going back and forth form VueJS Projects to vanillaJS and ExpressJS, I noticed some annoyances, like emmet not working, Syntax Highlighting broken, Snippets autocompletions not working or being weird. after some digging around I found that the Built-in Typescript features where disabled. This was because the VueJS Extension saying this:

> ⚠️ It's recommended to [use take over mode instead of VSCode built-in TS plugin](https://vuejs.org/guide/typescript/overview.html#volar-takeover-mode).

**—** [**Vue Language Features (Volar)**](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin)

I was looking for a way to have seperate settings for different VS Code Projects. Thats when I found out about Profiles.

The Profiles I currently have.

- ExpressJS
- MarkDown
- VanillaJS
- VueJS

### How I created a new Profile in vs-code

1. Click the Gear Icon in the Bottom Left
2. Create Profile…

![Image.png](Setup%20Profiles%20in%20VS-Code.assets/Image.png)

3. Use Current Profile

![Image.png](Setup%20Profiles%20in%20VS-Code.assets/Image%20(2).png)

4. Give the new Profile a Name

![Image.png](Setup%20Profiles%20in%20VS-Code.assets/Image%20(3).png)

5. Select/Deselect the Extensions for this Profile

![Image.png](Setup%20Profiles%20in%20VS-Code.assets/Image%20(4).png)

7. Click `Create Profile`
8. Done!

## Conclusion

Creating profiles in VS Code is a convenient and time-saving way to manage different types of projects. Instead of manually enabling or disabling extensions every time you switch between projects, profiles allow you to quickly switch between pre-configured setups.

### Resources

[Profiles in Visual Studio Code](https://code.visualstudio.com/docs/editor/profiles)

[Stack Overflow - Where Developers Learn, Share, & Build Careers](https://stackoverflow.com/)

