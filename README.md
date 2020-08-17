# GitHub Action to Add Release Metadata to The IOTA Tangle

This GitHub Action will take the contents of your GitHub tagged release and create an associated transaction on the IOTA Tangle. This way the data associated with the release becomes immutable.

The action will perform the following steps:

* Load the release tagged by the input `tag_name`
* Create a signature for the content of `tarball_url`
* Create a signature for the content of `zipball_url`
* Create a signature for each of the attached assets
* Attach a payload to the IOTA Tangle with information about the release and the hashes

### Inputs

- `tag_name`: The name of the tag for this release
- `comment`: An optional comment to include in the Tangle payload

### Outputs

- `tx_hash`: The hash of the transaction on the Tangle
- `tx_explore_url`: A url which can be used to explore the transaction on the Tangle


## Example Payload

```json
{
   "owner": "an-owner",
   "repo": "test-repo",
   "tag_name": "v0.1.0",
   "name": "First full release",
   "comment": "My Awesome Release",
   "body": "This is the body of the description.",
   "tarball_url": "https://api.github.com/repos/an-owner/test-repo/tarball/v0.1.0",
   "tarball_sig": "Me3ouGni0h50TOHQklopu3sJdLFh/ZVlPJom3aDRFVQ=",
   "zipball_url": "https://api.github.com/repos/an-owner/test-repo/zipball/v0.1.0",
   "zipball_sig": "jyQ8U1T4oMSEbT3e9NTuFyoMskwAvti3nmiYKtuh8LU=",
   "assets": [
      {
         "name": "attach-1.zip",
         "size": 150752,
         "url": "https://github.com/an-owner/test-repo/releases/download/v0.1.0/attach-1.zip",
         "sig": "YQylonV2i+5KtwVN0FxTU7ssWflX+6fC29COSbFOmfQ="
      },
      {
         "name": "attach-2.zip",
         "size": 153054,
         "url": "https://github.com/an-owner/test-repo/releases/download/v0.1.0/attach-2.zip",
         "sig": "a+Rgpf5gs0lpCJ8wt+eymkTdo99RbcP0o1PgLCIT2NE="
      }
   ]
}
```

## Usage

Create a GitHub workflow in you repo e.g. `/.github/workflows/tangle-release.yml`.
 Most of the environment variables are optional, except for the `IOTA_SEED` which must be 81 trytes in length. For more details on creating a seed see [IOTA Docs - Getting Started - Creating A Seed](https://docs.iota.org/docs/getting-started/0.1/tutorials/create-a-seed)

```yaml
on:
  push:
    tags:
      - 'v*' # Push events to matching v*

name: Create Immutable Release

jobs:
  build:
    name: Create Immutable Release
    runs-on: ubuntu-latest
    steps:
      - name: Tangle Release
        id: tangle_release
        uses: iotaledger/gh-tangle-release@v0.7.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          IOTA_SEED: ${{ secrets.IOTA_SEED }}
          IOTA_NODE: ${{ secrets.IOTA_NODE }} # Optional, defaults to https://nodes.iota.cafe:443
          IOTA_ADDRESS_INDEX: ${{ secrets.IOTA_ADDRESS_INDEX }} # Optional, defaults to 0
          IOTA_DEPTH: ${{ secrets.IOTA_DEPTH }} # Optional, defaults to 3
          IOTA_MWM: ${{ secrets.IOTA_MWM }} # Optional, defaults to 14
          IOTA_TAG: ${{ secrets.IOTA_TAG }} # Optional, defaults to GITHUB9RELEASE
          IOTA_EXPLORE_URL: ${{ secrets.IOTA_EXPLORE_URL }} # Optional, defaults to https://utils.iota.org/transaction/:hash
        with:
          tag_name: ${{ github.ref }}
          comment: My Awesome Release

```

## CLI

The core features are also available to use as a CLI independently of GitHub Actions.

If you install the npm package with:

```shell
npm install @iota/gh-tangle-release
```

You can then execute the following script.

```shell
gh-tangle-release
```

You will then be presented with the following options.

```shell
GitHub Tangle Release v0.7.0 🚀

Usage: gh-tangle-release [options]

An application for creating a transaction on the IOTA Tangle from a GitHub release.

Options:
  -v, --version               output the current version
  --github-token <string>     GitHub token for accessing your repository (required)
  --owner <string>            GitHub repository owner (required)
  --repository <string>       GitHub repository (required)
  --release-tag <string>      The release tag from the GitHub repository (required)
  --node <string>             Url of the node to use for attaching the transaction to the tangle (default:
                              "https://nodes.iota.cafe:443")
  --depth <number>            Depth to use for attaching the transaction to the tangle (default: "3")
  --mwm <number>              Minimum weight magnitude to use for attaching the transaction to the tangle (default: "14")
  --seed <string>             81 Tryte seed used to generate addresses
  --address-index <number>    Index number used to generate addresses (default: "0")
  --transaction-tag <string>  Tag to apply to the Tangle transaction (default: "GITHUB9RELEASE")
  --comment <string>          An optional comment to include in the Tangle transaction payload
  --explorer-url <string>     Url of the explorer to use for exploration link (default: "https://utils.iota.org/transaction/:hash")
  --help                      Display help


Example: gh-tangle-release --github-token a4d936470cb3d66f5434f787c2500bde9764f --owner my-org --repository my-repo --release-tag v1.0.1 --seed AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  ```
