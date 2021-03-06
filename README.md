# hyperGRC

hyperGRC is a lightweight, in-browser tool for managing compliance-as-code repositories in OpenControl format.

The goal is a low-profile, hyper-useful IT GRC tool supporting compliance-as-code practices beginning with managing reusable OpenControl files for information technology systems and components.

**hyperGRC uses a data format _mostly_ compatible with OpenControl. There are a few extensions to the OpeControl informal data specification. As OpenControl matures, hyperGRC will support if feasible.**

## Requirements

* Python 3.5+
* A few packages listed in `requirements.txt`

## Installation and Running

### Install and run hyperGRC from source

```sh
git clone https://github.com/GovReady/hyperGRC.git hypergrc
cd hypergrc
pip install -r requirements.txt

# Start hyperGRC
python -m hypergrc example/agencyapp
```

NOTES: 
* You may need to adjust the command for `pip` (.e.g `pip3`) depending on how Python 3 was installed on your system.
* Type CTRL+C to stop

### Install and run hyperGRC with virtualenv

Use virtualenv to keep the Python package dependencies for hyperGRC isolated from other Python software on your workstation. 

```sh
git clone https://github.com/GovReady/hyperGRC.git hypergrc
cd hypergrc
virtualenv venv -p python3
source venv/bin/activate
pip install -r requirements.txt

# Activate virtualenv
source venv/bin/activate

# Start hyperGRC
python -m hypergrc example/agencyapp
```
NOTES: 
* Type CTRL+C to stop
* Type `deactivate` to exit virtualenv

### Install and run hyperGRC with Docker

A `Dockerfile` is provided in this repository to launch hyperGRC in a Docker container. The `Dockerfile` is based on CentOS 7.

```sh
git clone https://github.com/GovReady/hyperGRC.git hypergrc
cd hypergrc
docker image pull centos:7
docker image build --tag hypergrc:latest .

# Start container with mounted volume (-v) and mapped ports (-p) in ephemeral mode (--rm) and interactive mode (-it)
REPOSITORY=`pwd`/example/agencyapp
docker container run -v $REPOSITORY:/opencontrol -p 127.0.0.1:8000:8000 --rm -it hypergrc:latest

# visit hyperGRC at `http://127.0.0.1:8000`
```

NOTES:
* Provide the container with access to an OpenControl repository on your workstation by mounting a volume using the docker `-v` option. Workstation path must be an [absolute directory](https://docs.docker.com/engine/reference/run/#volume-shared-filesystems) and container path must be `/opencontrol`. Above, we use `` `pwd` `` to help form the absolute path to the included example OpenControl files. `REPOSITORY` can be set to any absolute path on wokstation.
* Map a port on your workstation to the container using the Docker `-p` option, such as `-p 127.0.0.1:8000:8000`.
* Start hyperGRC in ephemeral `--rm` and interactive mode `-it` so that you can end it by typing CTRL+C.
* Visit hyperGRC at `http://127.0.0.1:8000`.

## Command-line options

### OpenControl repository paths

hyperGRC accepts several command-line arguments. You've already seen one: the local path to the OpenControl repository. You may specify one or more paths to OpenControl repositories to open them all up within hyperGRC.

```sh
python -m hypergrc example/agencyapp path/to/project2 ...
```

If you do not specify any paths on the command line, hyperGRC reads a list of paths to repositories from a file named `repos.conf`, e.g.:

```text
repos.conf
---------------
example/agencyapp
path/to/project2
```

Create this file if it does not exist if you would like to start hyperGRC without any command-line options. An example of such a file is in [repos.conf.example](repos.conf.example).

Start as:

```bash
python -m hypergrc
```

You may also specify files containing lists of paths to repositories on the command-line by preceding the listing file with an `@`-sign. The command above is equivalent to:

```bash
python -m hypergrc @repos.conf
```

### Other options

To bind to a host and port other than the default `localhost:8000`, use `--bind host:port`, e.g.:

```bash
python -m hypergrc --bind 0.0.0.0:80
```

## Understanding the compliance-as-code data files

OpenControl creates readable structured standard for representing component to control mappings. hyperGRC reads and writes OpenControl data YAML files, including:

* A system `opencontrol.yaml` file which containins metadata about the information technology system and lists the system's components and compliance standards in use.
* One or more `component.yaml` files which describe components of the information technology system. Each component has a name and other metadata and list of control implementations (i.e. control narrative texts).
* Zero or more `opencontrol.yaml` files for standards, i.e. lists of compliance controls such as NIST SP 800-53, NIST SP 800-53 Appendix J Priacy Controls, HIPAA, and so on.

A typical OpenControl repository contains files in the following directory layout:

```
├── opencontrol.yaml
├── standards
│   ├── opencontrol.yaml
│   ├── NIST-SP-800-53-r4.yaml
│   └── HIPAA.yaml
└── components
    ├── Component 1
    │   └── component.yaml
    └── Component 2
        └── component.yaml
```

Although not currently conformant with the OpenControl standard, hyperGRC also allows components to be broken out into multiple files:

```
...
└── components
    ├── Component 1
    │   ├── component.yaml
    │   ├── AC-ACCESS_CONTROL.yaml
    │   ├── SI-SYSTEM_AND_INFORMATION_INTEGRITY.yaml
    │   ...
    └── Component 2
        ├── component.yaml
        ...
```

For more details, see the files in example/agencyapp.

## Generating system security plans

### From the command line

hyperGRC includes a command-line tool to generate a partial system security plan in Markdown format. The tool concatenates all of the control narratives in an OpenControl system repository, adding headings and control descriptions.

For example, to generate a system security plan for the example application stored in this repository, run:

	python3 -m hypergrc.ssp -d example/agencyapp

The system security plan is printed to the console. It will look like:

```md
# Agency App Example System System Security Plan

# NIST SP 800-53 Revision 4

## SI: System and Information Integrity

### SI-3: Malicious Code Protection

> The organization:
>   a.  Employs malicious code protection mechanisms at information system entry
> and exit points to detect and eradicate malicious code;
>   b.  Updates...

##### OpenLDAP

Destruction configuration for developer access to organization-defined...
```

You will probably want to redirect the output to a file, e.g.:

	python3 -m hypergrc.ssp -d example/agencyapp > ssp.md

If you have [pandoc](https://pandoc.org/) installed, you could then convert the SSP into HTML or a Microsoft Word document:

```sh
pandoc -t html < ssp.md > ssp.html
pandoc -t docx ssp.md -o ssp.docx
```

The `-d` option instructs the SSP generator to include control descriptions. You may also add `--family XX` (e.g. `--family CP`) to output only controls for the given control family.

## Customizing project appearance

The appearance of each project can be customized by adding a css file called `_extensions/hypergrc/static/css/repo.css` to the project's repository and referencing the path to the `_extensions/hypergrc` directory in the `opencontrol.yaml` file like so:

```
# ...
standards:
  - ./standards/NIST-SP-800-53-r4.yaml
  - ./standards/NIST-SP-800-53-r4-privacy.yaml
certifications:
  - ./certifications/fisma-low-impact.yaml
_extensions:
  - ./_extensions/hypergrc
```

hyperGRC's includes `_extensions/hypergrc/static/css/repo.css` as the last css file loaded in the base template when the custom extension is specified in the `opencontrol.yaml` manifest and the file `repo.css` exists.

### Example project `repo.css` files

Customize project with a background color in project's.

```
/* Custom project styles */

body {
  background-color: rgb(247, 247, 247);
}
```

Customize project with a background image. Only URL loaded images are currently supported. Please respect creator's copyrights and only use properly-licensed images.

```
/* Custom project styles */

body {
  /*background-color: rgb(247, 247, 247);*/
  background: url("https://upload.wikimedia.org/wikipedia/commons/f/f7/Rocky_Mountain_National_Park.jpg") no-repeat center center fixed;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
}
```

## Development

Development is easier if hyperGRC is run in a way that it restarts when any source code changes occur, so that you can see your changes immediately. `nodemon` from the Node package manager is a handy tool to do that. [Install Node](https://nodejs.org/en/download/) [Mac OS X users first [read this](https://gist.github.com/DanHerbert/9520689)] and then run:

```sh
npm install -g nodemon
nodemon -e py -x python3 -m hypergrc
```

## Licensing

hyperGRC is copyrighted 2018 by GovReady PBC and available under the open source license indicated in [LICENSE.md](LICENSE.md).

