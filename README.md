# Release-Workflow

Here is a rough worflow for how software in the billinge group.

## Existing Projects

Existing projects should have a `rever.xsh` in the main directory.
If it does not please proced to New Projects.

To release the software you need to run `rever <new_version_number>`.
Rever should handle the version bumping, changelog creation, tagging, github release and conda-forge PR opening.
Once this is done you will need to merge the PR into the conda-forge feedstock (once the CI has passed).
The PR will be in the `https://github.com/conda-forge/<project_name>-feedstock` repo.
Please note that the rever command does not add new dependencies.
Rever also does not add new mantainers to the project
To add dependenceis or maintainers this you need to go edit the `rever/<project_name>-feedstock/recipe/meta.yaml` in the main directory.
Please see the [conda-forge](https://conda-forge.org/docs/index.html) docs for more information on how to edit the `meta.yaml` files.


## New Projects

To start a new project you need to add a `rever.xsh` file and add the project to conda-forge (for public projects).

### Rever

Here is an example `rever.xsh` file.
```python
$PROJECT = 'xpdtools'
$ACTIVITIES = ['version_bump',
               'changelog',
               'tag',
               'push_tag',
               'ghrelease',
               'conda_forge']

$VERSION_BUMP_PATTERNS = [
    ($PROJECT + '/__init__.py', '__version__\s*=.*', "__version__ = '$VERSION'"),
    ('setup.py', 'version\s*=.*,', "version='$VERSION',")
    ]
$CHANGELOG_FILENAME = 'CHANGELOG.rst'
$CHANGELOG_TEMPLATE = 'TEMPLATE.rst'

$GITHUB_ORG = 'xpdAcq'
$GITHUB_REPO = $PROJECT

$LICENSE_URL = 'https://github.com/{}/{}/blob/master/LICENSE'.format($GITHUB_ORG, $GITHUB_REPO)

from urllib.request import urlopen
rns = urlopen('https://raw.githubusercontent.com/xpdAcq/mission-control/master/tools/release_not_stub.md').read().decode('utf-8')
$GHRELEASE_PREPEND = rns.format($LICENSE_URL, $PROJECT.lower())
```

You will need to edit the:
  - `$PROJECT` to be the name of the project
  - `$GITHUB_ORG` to be the name of the github org
  - If the project is not part of the `xpdAcq` project please remove the `$LICENSE_URL ...` line and lower (since that inserts the xpdAcq specific license into the github release notes)


### Conda-Forge

To add the project to conda-forge first you need to create the first release (run `rever <version_number>`).

Then fork `https://github.com/conda-forge/staged-recipes`, add a new folder in recipes (with the project name as the folder name), copy the example `meta.yaml` into the new folder and modify it accordingly.
Please see the [conda-forge](https://conda-forge.org/docs/index.html) docs for more details on how to put in a recipe.
Once the PR is merged conda-forge will create a feedstock and package for your software (which will be installable via `conda install -c conda-forge <package_name>`).
