# Game Development Journal

A static blog styled as a printed newspaper / archival journal. Posts are Markdown files in `data/posts/`. No dependencies: only Node.js 18+ (https://nodejs.org).

## Project layout: code and data are separate
    data/                 EVERYTHING you write or upload (the editor reads and writes only here)
      posts/              blog posts and drafts (Markdown; `published: false` = draft)
      pages/              custom pages (About, ...), one JSON file each
      progress/           progress.json (the Progress page)
      site/               site.js (title, navigation, categories, footer, social links); pinned.json (Pinned Posts)
      media/images/       uploaded pictures (blog/<post>/, pages/<page>/); published at /images/
    src/                  site code: lib.mjs (rendering), social-icons.mjs, and the scripts the build inlines
    editor/               the Writing Desk (npm run editor)
    public/               the theme's own static files (style.css, fonts, paper textures, scripts); copied as they are
    tools/                tests and helper scripts
    build.mjs             the build (npm run build); writes the generated site to dist/
    dist/, .publish/      generated or temporary; safe to delete, never edited by hand
Image addresses in Markdown stay `/images/...`; the build copies `data/media/images` to `dist/images`. The build refuses to use `data/`, `src/`, `editor/`, `tools/` or `public/` as its output folder.

## The printed-paper look
The whole theme lives in `public/style.css` (commented in layers), with small texture files in `public/paper/` and the typeface in `public/fonts/` (Liberation Serif and Mono, SIL Open Font License, see `public/fonts/LICENSE-Liberation.txt`), so the site loads nothing from other websites.
- **Desk, sheet and slips**: a dark charcoal woven-cloth desk surface (generated SVG: `public/paper/fabric.svg` and `fabric-mottle.svg`), the newspaper sheet on it (fibres, grain, aging blotches, flecks, browned edges, subtly torn outer edges), and separate slips of paper (sidebar, code listings, handwritten notes).
- **Ink**: soft ink spread on text, an uneven-density print filter on the nameplate and big headlines, and tiny ink dropouts over everything printed. Photographs are left completely untouched: no filter, grain, tint or blending.
- **Torn edges**: `src/torn-edge.js` (inlined into every page by `build.mjs`) cuts the outline of the `.sheet > .paper` element. Most of the border is a clean, nearly straight paper edge; small irregular tears appear at random places (usually one or more on every edge), a few medium ones, and on roughly one page in five a single more noticeable tear. Tuning: the numbers in `makeModel` (how many, how wide, how deep) and `MAX_TEAR`. The texture simply continues up to the edge. It needs JavaScript; without it the edge stays straight.
- **Coffee marks**: `src/paper-stains.js` (also inlined by `build.mjs`) adds occasional pale dried-coffee rings (often incomplete) and tiny droplets in their own SVG layer, `.stains`. Most pages get one faint ring or none; marks are small, never close together, and keep clear of photographs, video and slips. The layer is painted *below* everything printed on the sheet (`z-index:-1` inside the sheet's own stacking context, see `style.css`), so it can never lie over a photograph, gallery, side-by-side row or video. Tuning: ring/drop counts in `build`, sizes in `makeRing`/`makeDrops`, colour `COFFEE` and the alpha values.
- **Randomness**: every page draws its own torn edge and coffee marks on every load (browser crypto randomness, nothing shared between pages). The pages are static HTML and both effects are added after load, so there is no markup to mismatch. Resizing never reshuffles them.
- **Layout**: dateline strip (volume, issue number, date, page number), nameplate, section bar, ruled columns, drop cap, justified hyphenated text, pull-quotes, a folio and page number in the footer. Page numbers follow the order Home, Blog, your pages, then articles oldest to newest.
- **Tuning**: colours are variables at the top of `style.css`. Texture strength is set in the SVG files (`inkloss.svg` = ink dropouts, `fibers.svg`, `grain.svg`, `blotch.svg` = aging).
- **Handwritten note**: in the editor choose Insert, Handwritten Note (or type `:::note`, your words, `:::`). It prints as a pencilled note on lined paper.
- **404 page**: `dist/404.html` is created on every build; GitHub Pages, Netlify and Cloudflare Pages show it automatically.
- **Editor theme**: the Writing Desk and Site Settings pages wear the same paper as the site. They load `public/style.css` (same variables, fonts, masthead, sheet and slips) and then `editor/editor.css`, which only styles the form controls and panels; `editor/theme.js` adds the torn edge and coffee marks by running the site's own `src/torn-edge.js` and `src/paper-stains.js` (the editor server hands those two files to the page). Pictures in the editor are never printed on: the ink-dropout texture is a *background* of the editor's sheet, not an overlay, and hidden sections are marked with a dashed border instead of being dimmed. Nothing in `public/` or `src/` is editor-specific.
- **Editor safety**: the Writing Desk only answers requests that come from its own page at `http://localhost:4321` (other websites open in your browser cannot read or change your files).

## Everyday use
    npm run editor
Open http://localhost:4321. **New Post**, write, **Insert Image** (choose file, add caption), **Save Draft** or **Publish**, then **Build Site**. Preview at http://localhost:4322. Pick an existing post from the left list to edit or delete it.

## Without the editor
Copy any file in `data/posts/`, change the frontmatter (`published: false` = draft), then run `npm run build`.

## Edit website text
Everything lives in `data/site/site.js`: title, subtitle, intro, navigation, footer, GitHub link, categories, and About page text. Mark one post `featured: true` to show it on the home page.

## Build and deploy
    npm run build
Upload the `dist/` folder to GitHub Pages, Cloudflare Pages or Netlify. For a GitHub Pages project site set `base: '/repo-name'` in `site.js` first.

## Push to GitHub (publishing from the editor)
**Push to GitHub** (top of every editor page) validates the site, builds it for GitHub Pages, shows what will change, and after you confirm commits and pushes the complete built website. Set it up once in **Site Settings > GitHub Publishing**: repository (pick from your repositories or paste the address), branch, folder (root or `/docs`), build folder, optional custom domain. **Change Repository** switches the target without touching your content.
- **Sign-in (nothing is stored by the editor).** Git uses the credentials you already have: `gh auth login` + `gh auth setup-git` (GitHub CLI), Git Credential Manager, or an SSH key. Optionally start the editor with `GITHUB_TOKEN` set to a fine-grained token limited to the repository (Contents: Read and write). It is passed to git through the environment of that one process only and is never written to a file, a commit or a message. To disconnect: **Disconnect** in the settings, `gh auth logout`, and/or remove github.com from your credential manager.
- **Base path.** Worked out automatically (`/repo-name` for project sites; none for `user.github.io` repositories or custom domains) and applied only to the published build; `site.js` is not changed. The editor's local preview is rebuilt afterwards.
- **Safety.** No force-push, ever. The editor works in a private clone in `.publish/` (git-ignored); GitHub's newer commits are kept; it only deletes files it published earlier (listed in `.writing-desk-manifest.json` inside the published folder); anything else in the repository is untouched. Builds containing broken links, missing images, `.env`/key files or token-like text are refused before anything is pushed. Success is reported only after GitHub itself reports the new commit.
- **GitHub Pages** must be set to *Deploy from a branch* (Settings > Pages) with the same branch and folder. The check in the settings page tells you if it is not.
- **Tests.** `npm run test:publish` runs the real server and real git against local stand-in repositories (no GitHub account needed).
- **Other tests.** `npm run test:editor` drives the Writing Desk's everyday workflows (saving, images and covers, Progress, Site Settings, Build, Delete) on a temporary copy of the project. `node tools/test-finder.mjs` tests article search and filtering. `npm run test:pinned` tests Pinned Posts, the Pinned Posts page, Expand / Collapse and the "Pin this post" box (editor, build and real browsers when Playwright is installed). All tests work on temporary copies and never touch your content. They refuse to run while your own editor is open (close `npm run editor` first), so they can never act on your real site.


## Inline images
1. In the editor, click in the text exactly where the image should go (between paragraphs, under a heading, anywhere).
2. Click **Insert Image** (or paste a screenshot with Ctrl+V) and choose a PNG, JPG, JPEG, WebP or GIF file.
3. Type an optional caption (printed as "Fig. 1.") and optional alt text, then **Insert at Cursor**.
The file is copied to `data/media/images/blog/<post-name>/` and a line like `![alt](/images/blog/post/shot.png "Caption")` appears in your Markdown. Identical files are never stored twice. Add `{px}` after an image for crisp pixel art. `:::gallery` ... `:::` and `:::compare` ... `:::` blocks still work.

## Cover image
In the **Cover Image** box click **Select Cover Image** (or **Change Cover Image**) and pick a file. It is copied into `data/media/images/blog/<post-name>/`, the post's `cover:` line is updated immediately, and a preview appears. When you change or remove a cover, the old file is deleted only if no post uses it anywhere (as a cover, inline image or gallery image). Covers never go into the article text, and inline images never change the cover.

## Deleting a post
Open the post, click **Delete**, then **Yes, Delete Permanently**. The `.md` file is removed, along with that post's images that no other post uses. Click **Build Site** afterwards so the article disappears from the website, search and archive.

## YouTube videos
Click in the text where the video should go, click **Insert YouTube Video**, paste a link (`youtube.com/watch?v=...`, `youtu.be/...` or `youtube.com/shorts/...`) and an optional caption, then **Insert at Cursor**. It is saved in the Markdown as:

    :::youtube
    https://www.youtube.com/watch?v=VIDEO_ID
    Optional caption on the next line
    :::

Only valid YouTube links are accepted, the player is a privacy-friendly `youtube-nocookie.com` embed that loads lazily, and raw HTML in Markdown is never rendered. Shorts get a portrait player.

## Site Settings
In the editor click **Site Settings** (top of the page). Edit the website title, subtitle, description, homepage text, navigation, About page, footer (including social links) and categories, watch the preview, then **Save Settings**. Everything is written to `data/site/site.js` (the only place this text lives), so it survives restarting the editor. Click **Build Site** to apply it to the website. Renaming a category also updates the posts that use it; a category that posts still use cannot be deleted.

## Pages, sections and navigation (Site Settings)
**Pages** tab: create any page you like (New Page), set its title, address (URL), whether it is published, whether its title is shown, and whether it appears in the main navigation (with its own navigation label). A page is a list of **sections**: add, rename, reorder (▲▼), hide (Visible), turn headings on or off, or delete them. Each section has a type: Markdown / mixed content (with Insert Image and Insert YouTube Video at the cursor), plain text, intro text, image, gallery, YouTube video, list, links or code. Pages are stored one file each in `data/pages/`.
**Homepage** tab: the homepage is also a list of sections (intro text, search box, featured article, latest posts, categories list, archive, or any custom section above) that you can reorder, rename, hide or remove, plus control over where each sidebar block appears.
**Navigation** tab: add, rename, reorder, hide or remove menu items; each points to the home page, the blog, any page, or an external link. Items point at the page itself, so changing a page address never breaks the menu.
Changing a page address changes its URL; links you typed by hand inside other content are not updated. Deleting a page removes it from the navigation and deletes only its own unused images, never blog posts.

## Several categories per post
In the post editor, pick any number of categories (each shows as a chip you can remove) or create a new one on the spot. Posts store `categories: ["Godot", "Animation"]`. Older posts with a single `category: "Godot"` line keep working and are only rewritten when you save them. A post appears under every one of its categories. Renaming a category in Site Settings updates the posts that use it; a category that posts still use cannot be deleted.

## Social links and footer icons
In **Site Settings > Footer**, below the footer text and GitHub link, the **Social Links** section lists each link with a name, a URL and an icon. The icon is picked automatically from the name or URL (GitHub, YouTube, Discord, X, Instagram, itch.io, Bluesky, Mastodon, Twitch, Kick, Reddit, LinkedIn, Dailymotion, Rumble, Codeberg, Behance, Dribbble, Buy Me a Coffee and more) and you can change it in the dropdown. If no icon fits, choose **Custom image** (upload a small square PNG/JPG/WebP/GIF) or **Text only** to show the name instead. Use ▲▼ to change the order. Everything is stored in `social` in `data/site/site.js` as `{ label, url, icon }` (plus `customIcon` for a custom image); older entries without an icon are given one automatically.
The footer shows the page navigation links in one row, with the social icons underneath. Each icon links to its URL in a new tab (`rel="noopener noreferrer"`) and has an accessible label and a tooltip. The GitHub link from the Footer tab is shown as the GitHub icon (not twice if it is also in the list). Icon artwork lives in `src/social-icons.mjs`; add a platform there and it appears in the dropdown. Any other `https://` link is accepted too: it gets the generic link icon (or a custom image / text label) and opens like the rest. Run `npm run test:social` to check the platform list, URL detection and the footer output.

## The Insert menu (images, videos, layouts, divider, table, code block)
Everything that is not plain text formatting is ONE button: **Insert** at the left of the toolbar (and above every Markdown section in Site Settings). It opens a menu in four groups; each choice opens the matching workflow and the result goes where your cursor is.

| Group | Options |
|---|---|
| Images | Single Image, Multiple Images, Image Gallery, Side-by-Side Images, Before / After |
| Videos | Single YouTube Video, Multiple YouTube Videos, Side-by-Side Videos |
| Mixed Media | Side-by-Side Image + Video (needs at least one of each), Custom Media Layout (any mix, you choose items per row) |
| Other Content | Horizontal Divider, Table (asks for columns and rows), Code Block (wraps selected text) |

The toolbar keeps only the buttons that format your selection (B, I, Heading, Subheading, Link, lists, Quote, Code) and the view toggles. The Cover Image box stays where it is because it sets the post's front-page picture rather than inserting text. Pasting a screenshot (Ctrl+V) opens Multiple Images with the picture ready.

All the media choices share one dialog (`editor/media.js`) that is just configured differently (see `MODES` there), so they behave the same:
- Choose many files at once (Ctrl/Shift-click in the file window; Single Image takes just one). Each gets a preview, a caption and alt text, ▲▼ reordering and Remove. Nothing is uploaded until you click Insert, so removed images never touch the disk. The same picture is never listed twice, even if renamed or picked again.
- YouTube links (watch, youtu.be, Shorts; several can be pasted at once, one per line) are checked by the same code the website build uses and shown with a thumbnail. An invalid link blocks the insert and says which one.
- Side-by-side layouts are saved in the Markdown as:

      :::row cols=2
      ![Alt text](/images/blog/post/a.png "Caption")
      [video](https://www.youtube.com/watch?v=VIDEO_ID "Caption")
      :::

  `cols=` (2 to 4) is optional. On the website the items sit in columns when there is room and stack on phones. Photos keep their proportions and nothing overflows.
- Limits and safety: PNG, JPG, JPEG, WebP and GIF only, up to 25 MB each. File contents are checked (a text file renamed `.png` is refused; a PNG named `.jpg` is stored as `.png`). Image and video addresses are validated and escaped when the site is built.
- Keyboard: the Insert button opens with Down arrow; Up/Down/Home/End move through the options; Escape closes it.
- Site Settings also has a **YouTube videos** section (any number of videos, reorderable) and an **Image gallery** section. Older single-video sections keep working and are converted when you save.

## Progress page
`/progress/` shows what is **Completed**, **In Progress**, **Paused** (started earlier, not finished, set aside for now) and **Planned**. New Planned items start at 0% and Completed items at 100%; both can be edited. In the editor open **Progress** (top navigation of the Writing Desk): add items to a section, edit title, description, status, percentage (leave empty for no progress bar), start, target and completion dates and an optional screenshot, reorder with ▲▼, move an item to another section with its Status list, or delete it. Click **Save Progress**, then **Build Site**. Everything is stored in `data/progress/progress.json` (the only source; no browser storage) and images in `data/media/images/pages/_progress/` (an image that is replaced, removed or belongs to a deleted item is deleted if nothing else uses it). The page's styling is in `public/progress.css`, which only reuses the variables and paper of `style.css`. The menu entry is a normal item in Site Settings > Navigation (link `/progress/`) and can be renamed, moved or hidden there.

## Pinned Posts
A small, quiet list of hand-picked articles for the **Home** and **Blog** pages, plus a page of its own. In the editor open **Pinned** (top navigation of the Writing Desk), or tick **Pin this post** in the post editor.
- **Pin / unpin / order.** On the Pinned page pick a post under *Other posts* and click **Pin**; **Unpin** puts it back. ▲▼ change the order (the website follows it exactly, everywhere). Click **Save Pinned Posts**, then **Build Site**. The limit is 50.
- **Pin this post (post editor).** The checkbox sits right above Save Draft / Publish and always shows whether the open post is pinned. Ticking or unticking it saves at once (no extra Save step) into the same `data/site/pinned.json`; a post is added at the end of the list, and the order of the other pins and the Home / Blog switches are never touched. On a brand-new post, tick it before the first save and it is pinned when the post is first saved. As with every change, click **Build Site** afterwards.
- **Where it shows.** Two independent switches, *Show Pinned Posts on the Home page* and *... on the Blog page*. Switching a page off only hides the section there; the pinned list is kept.
- **Collapsed by default.** On Home and Blog the section is a small, compact box that starts completely collapsed: you see only its heading, a small *Expand* control and *View All*. No pinned title, picture or preview is shown until the visitor clicks *Expand*; *Collapse* hides them again. Expanding lists every pinned post in the saved order. (Without JavaScript the box simply stays collapsed and *View All* still leads to the full list.)
- **The Pinned Posts page (`/pinned/`).** A small *View All* link in the heading row of the section on Home and Blog leads to it. It lists **every** pinned post in the saved order (thumbnail, title, date, excerpt), each linking to its original article. It is deliberately **not** in the header or footer navigation, and it does not depend on the two switches: it still lists everything while Home and Blog show nothing. With nothing pinned the address still works and says so (and the section and its link disappear from Home and Blog). The address `/pinned/` is reserved, so a custom page cannot take it. It gets the next page number after Progress, so no other page number changes.
- **Storage.** `data/site/pinned.json` holds only post addresses (in order) and the two switches. Articles are never copied, edited or given a "pinned" flag, so nothing else in them changes and nothing is duplicated. The file survives rebuilds and restarts (`npm run build` reads it); if it is missing, nothing is pinned. The editor, the preview and the production build all read this one file.
- **Unpublished or deleted posts.** The build leaves out a pinned post that is a draft or no longer exists, on Home, Blog and `/pinned/` alike (and the section disappears when nothing is left to show). The pin itself stays saved: publish the draft again and it returns in its place. The Pinned page marks such posts and lets you remove the pin.
- **On the website.** Each entry (once the box is expanded) is a small picture (if the post has a cover), the title and the date, linking to the original article. The box is deliberately a secondary, optional item: it is the last thing in the page's content area, below the main content **and** the sidebar (full width), on Home and on Blog (below the article list), never at the top. While you search, the search results replace the lists, the pinned section included. Its look lives in `public/pinned.css` (loaded only by pages that show the section or the Pinned Posts page, using only the colours, fonts and paper of `style.css`); to place the section elsewhere, change the `after:` option passed to `shell()` for Home / Blog in `build.mjs`.
