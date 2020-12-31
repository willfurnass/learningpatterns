# Learning Patterns blog

Thoughts by Will Furnass on research software, systems administration and teaching.

This static site is built using the [Hugo][hugo] static site generator and 
the [Terminal][hugo-theme-terminal] theme for Hugo.

## Building and serving the site on your own machine

 1. Install [Hugo][hugo].
 1. Clone this repo (including the [Terminal][hugo-theme-terminal] theme, which is a [submodule][git-submodule] of this repo):
    ```sh
    git clone --recursive git@github.com:willfurnass/learningpatterns-hugo.git learningpatterns-blog
    ```
 1. Build and serve:
    ```sh
    cd learningpatterns-blog
    hugo serve
    ```
 1. Browse to [http://localhost:1313](http://localhost:1313) to view the rendered site.

## License 

All content licensed under [Creative Commons Attribution-ShareAlike 4.0 (CC-BY-SA-4.0)][cc-by-sa-4-0].


[hugo]: https://gohugo.io
[hugo-theme-terminal]: https://themes.gohugo.io/hugo-theme-terminal/
[git-submodule]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[cc-by-sa-4-0]: https://creativecommons.org/licenses/by-sa/4.0/
