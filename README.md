# OvoEasy

Umbrella repository for the **OvoEasy** incubator project — an open-hardware system
for axenically raising oviparous vertebrates for experimental microbiome studies
(Trevelline Lab).

This repository ties the project's component repositories together as **git
submodules** and provides citable, version-pinned snapshots for each manuscript in
the series. Each branch pins the exact commit of every component used at a given
stage of the project, so a publication can reference one umbrella tag and have the
entire hardware/software state reproduced exactly.

## This branch: `dev` — next-generation OvoEasy

`dev` is the integration line for the **OvoEasy** redesign: the PCB, the controller
code, and the printed parts / assembly. It tracks the development tip of each
component. The previous-generation **Birdatron 9000** is retained here for now while
the redesign matures.

| Component | Repository | Description |
| --- | --- | --- |
| Assembly | [tanaes/OvoEasy_Assembly](https://github.com/tanaes/OvoEasy_Assembly) | Printed parts, 3D models, and assembly instructions |
| Code | [tanaes/OvoEasy_Code](https://github.com/tanaes/OvoEasy_Code) | Incubator controller code |
| PCB | _OvoEasy_PCB — to be added_ | KiCad PCB designs (incubator controller, light bar) |
| Birdatron 9000 | [tanaes/birdatron_9000](https://github.com/tanaes/birdatron_9000) | Previous-generation incubator (retained; see note) |

> **PCB:** the `OvoEasy_PCB` repository is not yet created. Once it exists it will be
> added here with `git submodule add https://github.com/tanaes/OvoEasy_PCB.git OvoEasy_PCB`.
>
> **Birdatron 9000:** retained as a submodule on `dev` for now. When `dev` is
> promoted to `main` for the OvoEasy publication, the Birdatron 9000 will be demoted
> from a submodule to a referenced citation (a link/footnote rather than embedded
> content), since it is the subject of a separate, earlier paper.

## Branches

| Branch | Purpose | Components |
| --- | --- | --- |
| `main` | First manuscript — Birdatron 9000 | `birdatron_9000` |
| `dev`  | Next-generation OvoEasy integration | `OvoEasy_Assembly`, `OvoEasy_Code`, `OvoEasy_PCB` (+ `birdatron_9000`, for now) |

## Cloning

```sh
git clone --recurse-submodules https://github.com/TrevellineLab/OvoEasy.git
cd OvoEasy
git checkout dev
git submodule update --init --recursive
```

Switching branches changes which components are present, so re-sync after a checkout:

```sh
git submodule update --init --recursive
```

## Releasing versioned snapshots for publications

A manuscript cites **one umbrella tag** and gets the complete, exact state of every
component reproduced. This works because each component is a **git submodule** — a
submodule records a single specific commit of the component, not "whatever is latest".

A release is a two-level freeze:

1. Freeze each **component** repo at the exact commit the paper uses (tag it there).
2. Point this umbrella's submodules at those commits, commit, and **tag the umbrella**.

The umbrella tag is the citable artifact.

### Tag naming conventions

| Level | Convention | Example |
| --- | --- | --- |
| Component repo | semantic version | `v1.0.0` |
| Component repo (paper-scoped, optional) | `<paper>-vX.Y` | `paper2-v1.0` |
| Umbrella | `<paper>-<focus>-vX.Y` | `paper2-ovoeasy-v1.0` |

Use **annotated** tags (`git tag -a`) so each tag carries a message, date, and author.

### Procedure

Run on the branch the manuscript publishes from.

**1. Freeze each component repo** — check out the exact commit and tag it:

```sh
cd <component-checkout>
git checkout <commit-or-branch>
git tag -a paper2-v1.0 -m "State used for <manuscript>, <date>"
git push origin paper2-v1.0
```

Repeat for every component on the umbrella branch you are releasing.

**2. Point the umbrella's submodules at those tags** — from the umbrella working copy:

```sh
git checkout dev                 # the branch you are releasing
for m in OvoEasy_Assembly OvoEasy_Code OvoEasy_PCB birdatron_9000; do
  git -C "$m" fetch --tags
  git -C "$m" checkout paper2-v1.0
  git add "$m"
done
git submodule status             # verify each pointer; a leading '+' means it drifted — re-add
```

**3. Commit and tag the umbrella:**

```sh
git commit -m "Freeze components for <manuscript> (paper2 v1.0)"
git tag -a paper2-ovoeasy-v1.0 -m "<manuscript> — frozen component set"
git push origin dev --follow-tags
```

**4. Cut a GitHub Release:**

```sh
gh release create paper2-ovoeasy-v1.0 \
  --title "Paper 2 — OvoEasy (v1.0)" \
  --notes "Frozen component set for <manuscript>."
```

**5. (Recommended) Archive to Zenodo for a DOI.** Enable the GitHub ↔ Zenodo
integration; publishing a Release creates a citable DOI.

> **Zenodo + submodules gotcha:** Zenodo's automatic archive captures only *this*
> repo's tree (the submodule **pointers**, not the component files). For a
> self-contained archive, build a recursive bundle and attach it to the Release:
>
> ```sh
> # pip install git-archive-all
> git-archive-all --prefix=OvoEasy-paper2-v1.0/ OvoEasy-paper2-v1.0.tar.gz
> gh release upload paper2-ovoeasy-v1.0 OvoEasy-paper2-v1.0.tar.gz
> ```

### Reproducing a published version

```sh
git clone --recurse-submodules https://github.com/TrevellineLab/OvoEasy.git
cd OvoEasy
git checkout paper2-ovoeasy-v1.0
git submodule update --init --recursive
git submodule status             # every component now at its frozen commit
```

### Notes

- **Where components live.** Submodules point at the `tanaes/*` repositories where the
  work is developed. If a component later moves into the `TrevellineLab` org, update
  its URL in `.gitmodules`, run `git submodule sync`, commit, and cut a new version.
- **Never cite a moving branch** — always tag. A submodule left on a branch drifts
  silently.
