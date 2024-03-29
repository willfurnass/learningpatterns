# Learning Patterns blog

A blog (plus other stuff) by Will Furnass on
research software, systems administration and teaching.

This static site is built using the [Hugo][hugo] static site generator and
the [Terminal][hugo-theme-terminal] theme for Hugo.

See [https://learningpatterns.me](https://learningpatterns.me) for the rendered version of this site.

## Building and serving the site on your own machine

 1. Install [Hugo][hugo].
 1. Clone this repo (including the [Terminal][hugo-theme-terminal] theme, which is a [submodule][git-submodule] of this repo):
    ```sh
    git clone --recursive git@github.com:willfurnass/learningpatterns.git learningpatterns
    ```
 1. Build and serve:
    ```sh
    cd learningpatterns
    hugo serve
    ```
 1. Browse to [http://localhost:1313](http://localhost:1313) to view the rendered site.

## Deployment

The site is automatically built and served by GitHub Pages.

Each push to the `main` branch will result in the site automatically being rebuilt.

The site can be accessed over HTTPS and HTTP via the `learningpatterns.me` and `www.learningpatterns.me` domains:

  * There are four `A` DNS resource record for `learningpatterns.me` that points at the IPv4 addresses for GitHub Pages.
  * There are four `AAAA` DNS resource record for `learningpatterns.me` that points at the IPv4 addresses for GitHub Pages.
  * There is a `CNAME` DNS resource for `www.learningpatterns.me` that points at `willfurnass.github.io`
  * GitHub Pages uses LetsEncrypt to automatically generate SAN TLS certificates for the domains.
  * GitHub Pages automatically redirects `www` to the apex domain.

## License

All content licensed under [Creative Commons Attribution-ShareAlike 4.0 (CC-BY-SA-4.0)][cc-by-sa-4-0].


[hugo]: https://gohugo.io
[hugo-theme-terminal]: https://themes.gohugo.io/hugo-theme-terminal/
[git-submodule]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[cc-by-sa-4-0]: https://creativecommons.org/licenses/by-sa/4.0/
