<!-- nnw-theme-identity:start -->
# NetNewsWire Theme Template

A starting point for making and publishing your own NetNewsWire theme.
<!-- nnw-theme-identity:end -->

This repository is a GitHub template. You make one repository from it per theme,
and that copy comes with:

- a starter theme to restyle;
- a live preview gallery that renders your theme in WebKit and publishes to GitHub
  Pages;
- a `check` command (and matching GitHub Actions workflow) that tests every case
  before you release; and
- a Publish workflow that packages the theme and creates the GitHub release.

The tooling lives in the [`nnw-theme`](https://github.com/dave-atx/nnw-theme) npm
package, which you run with `npx nnw-theme@2`. Nothing gets installed into your
repository, so tooling fixes reach your theme without a commit. You can do all of
this through a coding agent or run the commands yourself, and you don't need Xcode
or a NetNewsWire source checkout.

This is an independent community project; it isn't affiliated with or endorsed by
NetNewsWire. The theme format itself is documented in NetNewsWire's
[Themes technote](https://github.com/Ranchero-Software/NetNewsWire/blob/main/Technotes/Themes.md).

## Create your theme repository

Skip this if you're reading it in a repository you already made from the template.

On the [template's GitHub page](https://github.com/dave-atx/netnewswire-theme-template),
choose **Use this template → Create a new repository**, then clone your new
repository. Or, with the [GitHub CLI](https://cli.github.com/), create and clone it
in one step (replace `YOUR-THEME` with the name you want):

```sh
gh repo create YOUR-THEME --template dave-atx/netnewswire-theme-template --public --clone
cd YOUR-THEME
```

Use `--private` if you don't want the theme in the marketplace (it only lists public
repositories).

Don't fork the template or clone it directly. The marketplace skips forks, and a
plain clone still points at the template. `init` warns you if it finds a fork.

## With an agent

Give your coding agent this prompt, from your own repository or the template's
GitHub page:

> Help me make a NetNewsWire theme with `dave-atx/netnewswire-theme-template`.
> Guide me through setup and design. I want [describe the look or reading experience].

The agent should ask you to approve the theme's permanent identifier, and ask again
before it creates a GitHub release or changes anything else on GitHub.

## Without an agent

Steps 1 - 3 happen once. Step 4 is where you'll spend most of your time.

**1. Install the tools.** On macOS:

```sh
brew install node gh
gh auth login
```

You need Node.js 24 or later. `gh` is optional, though `init` uses it to fill in
defaults from GitHub, warn about forks, and join the marketplace. Without it, you
can join the marketplace by adding the `netnewswire-theme` topic in your repository
settings. On Linux, install the same tools however you normally would.

**2. Get your own copy.** See [Create your theme repository](#create-your-theme-repository)
above.

**3. Initialize and commit.** `init` asks for the theme's name, your name and home
page, and the theme's permanent identifier. It also offers to join the marketplace
and install WebKit; say yes to both unless you have a reason not to.

```sh
npx nnw-theme@2 init
git add -A && git commit -m "Initialize theme"
```

`init` only runs once, while `.nnw-theme-uninitialized` exists. If WebKit setup
failed or you skipped it, run `npx nnw-theme@2 setup` before step 5.

**4. Design.** Start the preview, then edit the files in your `.nnwtheme` folder
(mostly `stylesheet.css`). The gallery rebuilds every time you save; refresh the
browser to see the change. Ctrl-C stops it.

```sh
npx nnw-theme@2 preview
```

**5. Check.** Once it looks right, run the release check. It renders every case in
WebKit and offers to open the results when it's done. Fix any failures and run it
again.

```sh
npx nnw-theme@2 check
```

**6. Make the marketplace image.** This captures the Mac light-mode article at the
card's size and saves it as `screenshots/theme-preview.png`. Add `--fixture NAME` to
capture one of your own fixtures instead.

```sh
npx nnw-theme@2 screenshot --card --promote
```

**7. Commit and push.**

```sh
git add -A && git commit -m "Design my theme"
git push
```

GitHub runs the same check on every push. The first time, turn on **Settings → Pages →
Build and deployment → Source → GitHub Actions** so your gallery gets published.

**8. Publish.** See [Publish](#publish). For each later version, repeat steps 4 - 8.

The first `npx nnw-theme@2` downloads the package into npm's cache. Later runs reuse
it and pick up new 2.x releases on their own. The NetNewsWire rendering files the
preview uses ship inside the package, pinned and verified by hash.

### Shorter commands

`completion` prints shell completion for commands, options, and your fixture names.
It also defines an `nnw-theme` command (it runs `npx --yes nnw-theme@2`), so you can
type `nnw-theme check`:

```sh
# fish
npx nnw-theme@2 completion fish > ~/.config/fish/conf.d/nnw-theme.fish
# zsh (~/.zshrc, after compinit)
source <(npx --yes nnw-theme@2 completion zsh)
# bash (~/.bashrc)
source <(npx --yes nnw-theme@2 completion bash)
```

## What `check` tests

`check` validates and packages the theme, then renders 16 cases in WebKit: two
articles on macOS, iPhone, and iPad in light and dark, plus large text and Article
JavaScript turned off. Each fixture you add also gets checked on macOS and iPhone in
both appearances. Every footnote marker has to open NetNewsWire's popover with its
note, stay legible, and look like the others. External requests are blocked.

Results go to `build/preview/`, a gallery grouped by scenario with failures listed
first. The release ZIP goes to `build/release/`.

On the default branch, a passing check publishes that gallery to GitHub Pages. Pull
requests and failed checks keep it as a downloadable workflow artifact instead. You
need to turn Pages on once per repository (**Settings → Pages → Build and
deployment → Source → GitHub Actions**). If Pages already serves a site of yours from
a branch, the workflow leaves it alone and keeps the gallery as an artifact.

## Fixtures

Fixtures are the articles the preview and `check` render. Each one is a TOML file
whose keys are the template macros (`title`, `byline`, `feed_link_title`, `body`,
etc). The tool comes with two, `article` and `kitchen-sink`. Add your own in
`fixtures/` for content your theme needs to handle, and preview one with
`npx nnw-theme@2 render NAME`. A `fixtures/article.toml` or
`fixtures/kitchen-sink.toml` replaces the built-in one.

Real articles make the best fixtures. `npx nnw-theme@2 capture` explains how to save
the selected article from a NetNewsWire debug build, exactly as the app renders it.
Previews never touch the network, so web images show up as same-size placeholders.
To show real images, commit them under `fixtures/media/`.

A fixture can also spell out what its footnotes should do. `check` then verifies
each marker by its text, confirms the listed links stay ordinary links, and (with
`keep_with_word`) makes sure no marker wraps onto a line apart from its word:

```toml
[expect.footnotes]
plain_links = ["#not-a-footnote"]
keep_with_word = true

[expect.footnotes.notes]
"1" = "The first note's text."
"2" = "The second note's text."
```

Put these tables after the fixture's other keys, including `body`.
`npx nnw-theme@2 guide fixtures` lists every key.

## Theme Marketplace

The [NetNewsWire Theme Marketplace](https://dave-atx.github.io/nnw-theme-marketplace/)
finds themes by the `netnewswire-theme` GitHub topic, which `init` adds if you say
yes. Once your repository is public and its first stable release is out, the
marketplace picks it up. There's no form to fill out or pull request to open.

The marketplace card uses `screenshots/theme-preview.png`, so keep that file in
source control (gallery screenshots under `build/` stay ignored). `init` removes the
template's placeholder image; make your own with
`npx nnw-theme@2 screenshot --card --promote`. The screenshot workflow can open a
pull request when the image changes, if you turn on **Settings → Actions → General →
Workflow permissions → Allow GitHub Actions to create and approve pull requests**.

Joining is optional. To stay out, answer **No** during `init` (or have your agent
pass `--marketplace no`). If the topic is already there, remove it in your
repository's settings. Everything else (preview, packaging, publishing) works the
same either way.

## Publish

1. Run `npx nnw-theme@2 bump` and commit the changed `Info.plist`.
2. On GitHub, choose **Actions → Publish theme → Run workflow**.
3. Enter a tag starting with `v` (e.g., `v1.0.0`) and, optionally, release notes.

If you leave the notes empty and the repository has a [git-cliff](https://git-cliff.org)
config at `.github/cliff.toml`, the notes get generated from the commits since the
last tag.

The workflow only releases the current default-branch commit. It refuses a tag
that's already been used, a plist version that didn't go up, and any change to the
identifier or bundle name. Marketplace metadata is advisory, so private repositories
and ones without the topic can still publish. See `npx nnw-theme@2 guide publishing`
for more.

## What's in the repository

The theme is the one `*.nnwtheme` folder at the root:

- `stylesheet.css` is the visual design, with clearly named variables.
- `template.html` is the article structure and NetNewsWire macros.
- `Info.plist` holds the theme's public identity and integer version.

Next to it are `screenshots/theme-preview.png`, the optional `fixtures/` folder, and
a few small files that hook the repository up to the tooling: `AGENTS.md`, the agent
skill in `.agents/skills/`, and four workflows in `.github/workflows/` that call
`nnw-theme`'s reusable workflows. Each of those files starts with an
`nnw-theme-stub` marker, and `check` tells you when a newer version is available.

## Commands

```text
npx nnw-theme@2 init                  Personalize a fresh copy of the template (once)
npx nnw-theme@2 setup                 Install the WebKit browser for checks
npx nnw-theme@2 preview [--no-open]   Live gallery for editing; rebuilds on save
npx nnw-theme@2 render [fixture ...]  Write the gallery once, without checks
npx nnw-theme@2 check [--no-open]     Release check: package and test every case
npx nnw-theme@2 screenshot --promote  Check one case and make it the marketplace image
npx nnw-theme@2 capture               Explain how to capture a real article as a fixture
npx nnw-theme@2 package               Validate and build the release ZIP only
npx nnw-theme@2 bump [--yes]          Increase the integer theme version
npx nnw-theme@2 marketplace enable    Add the GitHub discovery topic
npx nnw-theme@2 guide [topic]         Read the authoring guide
```

[AGENTS.md](AGENTS.md) has the rules agents follow in this repository. Report
tooling problems at [dave-atx/nnw-theme](https://github.com/dave-atx/nnw-theme/issues).

## License

The template's files and documentation are Apache-2.0. The starter theme is 0BSD, so
you can replace its identity and license without carrying a notice forward.
