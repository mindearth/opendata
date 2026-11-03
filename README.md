# Mindearth Open Data — STAC Catalog

A [STAC](https://stacspec.org/) catalog of the open datasets released by
[Mindearth](https://mindearth.ai). The data is hosted on
[Source Cooperative](https://source.coop/mindearth). This repository holds only
the metadata, which describes each dataset and points to its cloud-native files.

## Datasets

Each dataset is a STAC Collection in `collections/<dataset-id>/collection.json`.
Each collection includes the dataset's description, extent, license, providers
and data assets, plus links to its Source Cooperative page and documentation.

To see which datasets are available, browse the [`collections/`](collections/)
folder, follow the `child` links in [`catalog.json`](catalog.json), or list them
from Python (see [Usage](#usage)).

## Structure

```
catalog.json                          # root Catalog, one "child" link per dataset
collections/
  <dataset-id>/collection.json        # one Collection per dataset
```

This is a self-contained static catalog: every internal link is relative, so it
works wherever it is served (GitHub, a static web host, or a local clone).
Cloud-native stores such as Zarr are described as collection-level assets. Items
are only used when a dataset is split into separate files, such as one file per
tile or per date.

## Usage

### Browse

Open the catalog in [STAC Browser](https://radiantearth.github.io/stac-browser/)
by pasting the raw URL of `catalog.json`.

### Python

```python
import pystac

catalog = pystac.Catalog.from_file("catalog.json")  # or the raw URL
for collection in catalog.get_collections():
    print(collection.id, collection.title)
    for key, asset in collection.assets.items():
        print("  ", key, "→", asset.href)
```

Each collection's `describedby` link points to the dataset's README, which
explains how to read the data.

## Adding a new dataset

New datasets are added only through pull requests. Don't push directly to
`main`.

1. **Create a branch** from `main`, for example `add-<dataset-id>`.
2. **Add the collection** at `collections/<dataset-id>/collection.json`. The
   easiest way is to copy an existing collection and edit it. Make sure it has:
   - a unique `id` that matches the folder name
   - `title`, `description`, `keywords` and `license`
   - `extent`: WGS84 bbox and temporal interval
   - `providers`: at least Mindearth and Source Cooperative (`host`), plus any
     upstream producers
   - the data asset(s): `href`, `type`, `roles`, and `proj:*` fields where
     relevant
   - relative `root` and `parent` links to `../../catalog.json`
   - `license`, `about` (Source Cooperative page) and `describedby` (README)
     links, plus `cite-as` if the dataset has a DOI
   - every extension it uses listed in `stac_extensions`
3. **Link it from the root catalog** by adding a `child` link to
   `catalog.json`:
   ```json
   {
     "rel": "child",
     "href": "./collections/<dataset-id>/collection.json",
     "type": "application/json",
     "title": "<Dataset title>"
   }
   ```
4. **Validate** the catalog locally:
   ```bash
   uv run --no-project --with 'pystac[validation]' python -c \
     "import pystac; c = pystac.Catalog.from_file('catalog.json'); c.validate(); [x.validate() for x in c.get_children()]; print('ok')"
   ```
5. **Open a pull request** against `main`. Include the Source Cooperative link
   and a short description of the dataset. A reviewer checks the metadata
   against the published data and merges the pull request once validation
   passes.

You don't need to edit this README when you add a dataset.

## Licenses and attribution

The metadata in this repository is released under
[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/). Each dataset
has its own license and attribution requirements, given in its collection's
`license` field and `license` link and in the dataset's README.

## Contact

Mindearth: <https://mindearth.ai>
