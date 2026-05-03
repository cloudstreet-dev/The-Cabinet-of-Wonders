# The Cabinet of Wonders

An encyclopedia of techniques, results, and phenomena that work despite seeming like they shouldn't.

A CloudStreet book.

## What this is

Each entry leads with the holy-shit hook — the result stated plainly, before any setup — and then explains the actual mechanism as deeply as it can without flattening the awe. The reader is a competent developer who has shipped real work; the prose assumes that. Wonder is preserved by accuracy and depth, not by adjectives.

The cabinet spans mass point geometry, zero-knowledge proofs, Bloom filters, Rowhammer, GPS's relativistic clock correction, the Y combinator, Banach–Tarski, and dozens more. It also revisits the ambient magic we forgot was magic — the things you stop noticing once you ship them.

## CloudStreet

CloudStreet publishes books written by AI systems that take the engineering reader seriously. No motivational register, no hype, no metaphors that strain. The contract is simple: tell the truth about how the thing works, and trust the reader to feel the wonder.

## Authorship

This book was written by Claude Code Opus 4.7 High. Every word, every derivation, every code sample. The byline is honest and not decorative. Errors are mine; corrections via pull request are welcome and will be merged into the canonical text.

Georgiy Treyvus, CloudStreet's Product Manager, maintains the book backlog and shaped the entry list.

## Reading locally

```sh
cargo install mdbook
git clone https://github.com/cloudstreet-dev/The-Cabinet-of-Wonders.git
cd The-Cabinet-of-Wonders
mdbook serve --open
```

## Deployment

Pushes to `main` build the book with mdBook and deploy to GitHub Pages via `.github/workflows/deploy.yml`. The live edition mirrors `main`.

## License

CC0 1.0 Universal. See `LICENSE`. Take it, fork it, rewrite it, embed it in your own work. Attribution is appreciated but not required.
