MANAGING MOBIO

Mobio is best managed with a Docker-managed toolkit called the Ontology Development Kit. Once loaded, ODK is pretty low profile - it only comes into action when one is at the command prompt in the mobio repository's /src/ontology folder, by running the "> sh run.sh" command.  That accesses (on demand) the robot program and other software that is tucked inside the docker container, operating in the context of the current directory. Once complete, one is dropped back into the same directory.

Instruction resources that may be helpful for more details: 

	https://oboacademy.github.io/obook/howto/odk-create-repo/
	https://oboacademy.github.io/obook/tutorial/setting-up-project-odk/
	http://pato-ontology.github.io/pato/odk-workflows/ReleaseWorkflow/

Final folder structure on contributor's computer typically at:

../Documents/GitHub/mobio/ 				Github repo on your computer, also where initial ODK/docker config files can be added.

../mobio/mobio.owl 						Main ontology file
   ...
../mobio/src/ontology/					Where mobio editable files are located
../mobio/src/ontology/mobio-edit.owl   	Main editable mobio file
../mobio/src/ontology/imports/			Import files from other ontologies

GET DOCKER AND ODK

For all users, to load Docker and the ODK:

Install Docker into your computer (https://www.docker.com/get-docker). 
It doesn't matter what folder you are currently in. 
To test, type in command line:

	> docker ps

1) Get docker to load latest ODK version image for reuse. 
   DO THIS OCCASIONALLY TO UPDATE odkfull VERSION!!!

	> docker pull obolibrary/odkfull


______________________________________________________________________________
UPDATING mobio-edit.owl or IMPORT FILES

Similar instruction: http://pato-ontology.github.io/pato/odk-workflows/EditorsWorkflow/

	> cd src/ontology

OVERALL PROCESS

1) Create a GitHub branch to place your changes on

Edit and save mobio-edit.owl or whichever ontology import file you need.

2) TEST BUILD

	> sh run.sh make IMP=false test

Note that "> sh run.sh" involkes ODK via Docker, but lets you stay on command line and not have to do anything more to get access to ODK commands.

Usually logical errors and accidental spaces in new term URLS etc must be dealt with. While you might see a few errors reported on screen, the full report containing these is at:

	> less reports/mobio-edit.owl-obo-report.tsv
	
3) Commit changes to GitHub branch

4) Make pull request on that branch

Caretakers of the GitHub mobio repository will provie feedback and or approve pull request merge into master (main) branch.


NEW IMPORT FILE TERM (continued from above)

Similar instruction: http://pato-ontology.github.io/pato/odk-workflows/UpdateImports/#refresh-imports

1) Ensure id of new term is entered in respective /src/ontology/imports import file (having .txt suffix):

	bfo_terms.txt
	go_terms.txt
	ro_terms.txt
	so_terms.txt
	...

2) Then to refresh [import file].owl output:

	> sh run.sh make imports/[name of import ontology]_import.owl
	

One exception at moment seems to be RO, which has a number of branches generated even though just a few 
3) Now one will be able to see new term in Protege when opening mobio-edit.owl  Sometimes the ontology file has to be reloaded though if Protege is already open to it.

A detail with import files is that they import terms from a local mirror of an ontology, large or small.  Sometimes these mirrors have to be reloaded due to changes on live repositories, which following command does. Refreshing local mirrors may take more memory though.

	> sh run.sh make no-mirror-refresh-merged


NEW IMPORT FILE SOURCE

1) Add new xxx_terms.txt file to src/ontology/imports/ foldermobio-odk.yaml.  
2) Then refresh the install file src/ontology/mobio-odk.yaml with new term source.  
3) Then refresh configuration:

	> sh run.sh make update_repo

ROBOT FILES

TO DO - EDIT AND TEST THIS:

Imports are by default assumed to be driven by "SLME" - Syntactic Locality Module Extractor (see http://robot.obolibrary.org/extract).  One can change this configuration to use other kinds of import, using "module_type: custom", OR using the https://robot.obolibrary.org/template command that extracts terms from spreadsheet / tsv tables, which is done via following pattern in rc/ontology/mobio-odk.yaml: 

  products:
    - filename: mycomp.owl
      use_template: TRUE
      template_options: --add-prefixes [various configuration options] 
      templates:
        - template1.tsv
        - template2.tsv
        - 

TESTING CHANGES TO MOBIO

	> sh run.sh make IMP=false test

______________________________________________________________________________
CREATE NEW RELEASE/PUBLISH

Similar instruction: http://pato-ontology.github.io/pato/odk-workflows/ReleaseWorkflow/

There are two steps to this.

1) Create a branch dedicated to release.

2) Do the build:

	> sh run.sh make prepare_release -B

Can take a few minutes.

3) Commit changes to branch.

4) Create GitHub Release with version info.

OCCASIONALLY UPDATE ODK

Similar instruction: https://oboacademy.github.io/obook/howto/odk-update/

	> cd ~Documents/Github/mobio/src/ontology/
	> docker pull obolibrary/odkfull

Do this twice:

	> sh run.sh make update_repo
	> sh run.sh make update_repo

And technical note, edit: 

	> nano .github/workflows/qc.yml

Edit container line to point to odkfull version.

	container: obolibrary/odkfull:v1.3.0


______________________________________________________________________________
INITIAL CONVERSION TO LATEST ODK

Here we create a new Repository with the Ontology Development Kit, then weave in existing mobio ontology files.

1) Do the GET DOCKER step above.

2) Create github "working" folder for mobio project, and navigate to it

	> cd ~/Documents/GitHub/
	> md mobio
	> cd mobio

3) In that folder make project.yaml file for ODK setup to read and implement. This will be converted to src/ontology/mobio-odk.yaml
 
id: mobio
title: Mobilome Ontology
github_org: arpcard
repo: mobio
release_artefacts:
  - base
  - full
  - simple
primary_release: full
import_group:
  products:
    - id: bfo
    - id: ro
    - id: so
    - id: go

4) Download the wrapper script (using wget for Macs)

	> wget https://raw.githubusercontent.com/INCATools/ontology-development-kit/master/seed-via-docker.sh 

5) Run the wrapper script, using project.yaml

	> sh ./seed-via-docker.sh -C project.yaml

or for a fresh try (removes old content):

	> sh ./seed-via-docker.sh -c -C project.yaml

This creates a target/mobio/ folder.  
If all is well, copy it up to mobio/ github folder. 
NOTE: this includes HIDDEN .git, .github and .gitignore folders and files.  Copy up the .github and .gitignore folders.  The .git one shouldn't replace an existing Github repo like mobio though.

7) Create fresh Mobio from existing (old install) mobio-edit and ./imports folder

	- Adjust src/ontology/imports/ files to have ids from old install

______________________________________________________________________________
SETTING UP CONTINUOUS INTEGRATION

TO DO: http://pato-ontology.github.io/pato/odk-workflows/ContinuousIntegration/

