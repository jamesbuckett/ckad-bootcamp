# Contributing

Thanks for your interest in improving this CKAD bootcamp tutorial. Contributions of all sizes are welcome — typo fixes, clearer explanations, missing examples, or new sections.

## How to contribute

- **Spot a bug or typo?** Open an issue, or send a PR directly. Small fixes don't need a prior issue.
- **Want to add or restructure content?** Open an issue first so we can agree on scope before you write the section.
- Fork the repo, create a branch, push your changes, and open a pull request against `master`.

## Testing examples

All snippets are designed to run on a single-node cluster. They are developed and tested against [Docker Desktop](https://www.docker.com/products/docker-desktop). If you change a YAML manifest or `kubectl` command, please apply it locally first to confirm it works end-to-end.

Each chapter ends with a **Clean Up** section. Run that before moving to the next chapter to avoid leftover resources interfering with later examples.

## Style conventions

The repo uses a few patterns consistently — please follow them when editing:

- **Chapter title:** `# Kubernetes Tutorial - <Topic>`.
- **Sections** use the `<details><summary>...</summary><p>` collapsible HTML pattern. This renders as a click-to-expand block on GitHub.
- **Code fences** are always language-tagged: `bash`, `yaml`, `console`, `diff`. Use lowercase tags.
- **Inline call-outs** in YAML examples use the `#👈👈👈` marker followed by a short note pointing at the line of interest.
- **Problem framing** uses the pattern:
  ```
  > Problem Statement: <user-facing need>
  >
  > tl;dr – <one-line summary>
  ```
- **Upstream references** use `kubernetes.io bookmark: [Title](url)`.
- **Section ends** with `_End of Section_` and a `Next [Chapter Name](url)` link.

## Images

Images are hosted on GitHub's user-content CDN (`user-images.githubusercontent.com`). When adding new diagrams, attach them to a GitHub issue or PR comment to get a stable URL, then reference that URL from the markdown.

## Licensing

By contributing, you agree that your contributions will be licensed under the same [LICENSE](LICENSE) as the rest of the repo.
