---
layout: post
title: "Distributing a Flask Application to a Secure Offline RHEL Box with RPM"
description: >
  How to create a dist mechanism for Python WebApp (Flask) to a secure, offline
  destination box using RPM.
categories: coding, devops
tags: [python, flask, web, linux, rpm]
comments: true
image: /assets/img/blog/general/python-flask.png
---
> This page explores the creation of a distribution mechanism for Python
WebApp (Flask) app to Linux RedHat environment that has no Internet access
due to security constraints, using RPM packaging tool.
{:.lead}

- Table of Contents
{:toc}

## The Burden of Python Apps distribution

Python is growing every year as one of the most used programming languages.
It's a very productive language and very pleasant to work, due to its powerful
built-in types, clean syntax, simple package structure, and ability to work
with both object-oriented and procedural paradigms, amongst other
characteristics.

However, one thing that is not that easy is the distribution part. Java
programmers as myself are used to consider distribution as a natural, simple
part of the development lifecycle. In Java, artifacts are packaged as
__Jars__, that can run in virtually any single __Java Virtual Machine__,
regardless of the operational system being adopted.

In Python is different. Since the concept of "Virtual Machine" does not exist
as it happens with Java, there is no unique way to package your artifacts for
deployment, because one must consider the target server characteristics in
order to elaborate a distribution strategy. Additionaly, Python community
created many different ways to package and deploy artifacts. This gives
flexibility to the developers, but at a cost of additional overhead.

This article describes a procedure to cover a very specific but very common
scenario for many developers: to distribute a Python Flask application
created in Windows 10 to a secure Linux Red Hat Environment, that
**has NO internet access** (due to security policies). The artifact to
be generated and delivered will be a RPM that already contains all its
required dependencies.

## Requirements

This article assumes that a programmer has the following scenario in your
Windows workstation:

- a [Flask](https://flask.palletsprojects.com/en/1.1.x/) App application,
adopting a virtual environment with [pipenv](https://pipenv.pypa.io/en/latest/)
and `Python >= 3.6`.
- a `Windows Git Client` (for git bash usage)
- an `Oracle Virtual Box` configured with `CentOS 7` Virtual Machine
(required to perform the RPM creation) and `Python >=3.6`. This machine
must have Internet access.

## Quality Report: Our Hypothetical Application

The best way to explain the dist mechanism is to consider a hypothetical
application structure. Let's assume that our application's name is
**quality_report**, and it has the folowing basic structure:

```tree
.
|-- Pipfile
|-- README.md
|-- config.yaml
|-- dist
|   |-- generate-dist-package.sh
|   `-- rpmbuild
|       |-- BUILD
|       |-- BUILDROOT
|       |-- RPMS
|       |-- SOURCES
|       |   `-- quality_report-0.0.1
|       |-- SPECS
|       |   `-- quality_report.spec
|       `-- SRPMS
|-- setup.py
`-- src
    |-- some_module
    |-- static
    |   |-- css
    |   |-- images
    |   `-- javascript
    `-- templates
```

Let's cover each one of these elements in details.

### README.md: documentation

This is a default markdown file for project documentation.

### Pipfile: where dependencies' configuration take place

This is a required file for [pipenv](https://pipenv.pypa.io/en/latest/).
It declares the dependencies of the application, and it is similar to the
following:

```python
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = false

[dev-packages]
pytest = "*"

[packages]
flask = "*"
objectpath = "*"
confuse = "*"
requests = "*"
pyopenssl = "*"
```

This is a [Flask](https://flask.palletsprojects.com/en/1.1.x/) application,
that uses some minor dependencies for configuration and JSON parsing.

### config.yaml: defining our properties

This is a configuration file, used to externalize application's internal
properties. It is only required if [confuse](https://confit.readthedocs.io/en/latest/)
is being used as a configuration mechanism. The hypothetical content of this
YAML file is the following:

```yaml
app_name: quality_report

backend_api_url: https://quality_report:443/

headers:
  api_auth_token: 11223345-1111-11b1-33n3-123hj32332hj  
```  

### setup.py: defining what is going to be deployed

This file defines the artifacts of our [Flask](https://flask.palletsprojects.com/en/1.1.x/)
app that will be distributed.

```python
import setuptools

# use README as app description
with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="quality_report",
    version="0.0.1",
    author="Daniel",
    author_email="daniel@arneam.com",
    description="Quality Report application package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/medeiros/quality-report",
    packages=setuptools.find_packages(),
    package_data={
      'src': ['static/css/*', 'static/images/*', 'static/javascript/*',
              'templates/*']
    },
    data_dir=[
        "config.yaml"
    ],
    install_requires=[
        'flask', 'objectpath', 'confuse', 'requests', 'pyopenssl'
    ],
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)
```

The static and configuration files must be declared in specific sections, as
well as the required dependencies. The packages (python code) are also
described, but it is done automatically by the `setuptools.find_packages()`
method.

### dist dir: structure for distribution

This dir contains the `rpmbuild` structure (dir template for RPM creation),
along with `shell` and `spec` files, to be explained in the following.

#### generate-dist-package.sh: creating our package for distribution

This file is responsible to package the Windows code of our application, along
with the required versions' definitions. It will not package the
dependencies, but only the information (version) about them.  

```bash
#!/bin/bash

#  remove old dist artifacts
$ rm -rf build *.egg-info dist/*.whl dist/*.gz

# generate a pipenv lock for artifact freezing, in a text file
$ pipenv lock -r > requirements.txt

# This will generate a wheel `.whl` file in `dist/` directory.
# Setup.py is the file that defines the static and config files to be packaged,
#   along with required dependencies and python version required.
$ python setup.py bdist_wheel  

# package the required artifacts for distribution
$ tar zcvf ./dist/quality_report-dist-0.0.1.tar.gz dist/*.whl \
    dist/rpmbuild dist/generate-rpm-package.sh requirements.txt config.yaml

# remove the no longer required wheel file
$ rm -rf build *.egg-info dist/*.whl
```

As we can see, the dependencies' freeze happened and the deps/version data was
saved in a `requirements.txt` file. This file, along with the `config.yaml`
file and the `generate-rpm-package.sh` file were grouped and packaged in a
`.tar.gz` file, together with a wheel file (that is the file that contains
all the Python package code).

The content of a `requirements.txt` file would be similar to the following:

```text
-i https://pypi.org/simple

--trusted-host pypi.org

certifi==2020.6.20

cffi==1.14.3

chardet==3.0.4

click==7.1.2; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4'

confuse==1.3.0

cryptography==3.1.1; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4'

flask==1.1.2

idna==2.10; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3'

itsdangerous==1.1.0; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3'

jinja2==2.11.2; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4'

lxml==4.6.1; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4'

markupsafe==1.1.1; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3'

objectpath==0.6.1

pycparser==2.20; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3'

pyopenssl==19.1.0

pyyaml==5.3.1

requests==2.24.0

six==1.15.0; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2'

urllib3==1.25.11; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4' and python_version < '4'

werkzeug==1.0.1; python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4'
```

#### generate-rpm-package.sh: generating a RPM in the CentOS machine

This file is responsible to generate a RPM file based on the `.tar.gz` file
created by the `generate-dist-package.sh` shell script.

```bash
#!/bin/bash

## preparation
# to make sure that the following programs exist in the CentOS7 environment
yum install gcc rpm-build rpm-devel rpmlint make python3 python3-devel \
  bash coreutils diffutils patch rpmdevtools dos2unix -y

## execution
## create an app directory to group all artifacts/deps and RPM'em

## download the requirements.txt file dependencies' and save them in
##   the app directory
mkdir -p ./quality_report-0.0.1 && pip3 download -r ../requirements.txt \
  -d ./quality_report-0.0.1
## move all code artifacts to the same app directory
cp -R quality_report*.whl ../requirements.txt ../config.yaml \
  ./quality_report-0.0.1

## create a .tar.gz to package all app artifacts from the directory
## this package will be saved in the RPM SOURCE structure
tar -zcvf rpmbuild/SOURCES/quality_report-0.0.1.tar.gz ./quality_report-0.0.1

## extract a .tar.gz file in the RPM SOURCE directory
cd rpmbuild/SOURCES
tar xvf quality_report-0.0.1.tar.gz

## generate the RPM package based on the SPEC file and the SOURCE code
cd ..
rpmbuild --define "_topdir `pwd`" -v -bb SPECS/quality_report.spec
```

### quality_report.spec: RPM specification file

This file explains to the RPM how to install our application. It requires - and
will ask for - some dependencies in the environment (`python3`, `python3-devel`).
It will create a directory in the `/opt` directory, deploy the app code on in,
and install the application as a systemd service.

In the following code, please replace all the `\%` by `%`, removing the
trailing `\`. This slash was added here in order to escape the `%`, due to
incompatibilities with Jekyll (the mechanism that renders this page).
{:.note}

```spec
Name:           quality_report
Version:        0.0.1
Release:        1\%{?dist}
Summary:        This application generate reports for quality analysis.

License:        unspecified
URL:            https://example.com/\%{name}
Source0:        https://example.com/\%{name}/release/\%{name}-\%{version}.tar.gz

BuildRequires:  bash, systemd, python3, python3-devel
Requires:       bash, systemd, python3
Buildroot:      /opt/\%{name}

\%description

QUALITY_REPORT v0.0.1
---------------------------------------------------------------------
Some context goes here.

Installation
---------------------------------------------------------------------
Explain installation steps here.

This solution is installed by a RPM package, that creates the following files:

    - /opt/quality-report/ (application directory)
    - \%{_unitdir}/quality-report.service (service config file)

This files can be also listed with the following command:

$ rpm -ql quality-report

In order to start the application as a service, the standard commands may be
used:

    systemctl enable quality-report
    systemctl start quality-report

This service is configured to start automatically in case of system reboot.

Logging
---------------------------------------------------------------------
The logs are turned off.
Once they're turned on (uncommenting the shell script 'echo lines'), then
the log would be visualized by running:

    journalctl -u quality-report -f


\%global _python_bytecompile_extra 0

\%prep
\%global _python_bytecompile_extra 0
\%global _python_bytecompile_errors_terminate_build 0
\%define debug_package \%{nil}  # in order to prevent debug packages errors
\%setup -q

\%build
cat > \%{name}.service <<EOF
[Unit]
Description=quality-report

[Service]
Environment=QUALITYREPORTDIR=/opt/\%{name}
Environment=FLASK_APP=/opt/\%{name}/src/app.py
Environment=PYTHONPATH=/opt/\%{name}/lib
ExecStart=/usr/bin/python3 -m flask run -h 0.0.0.0 -p 9091 --cert=adhoc
WorkingDirectory=/opt/\%{name}

[Install]
WantedBy=multi-user.target
EOF

\%install
mkdir -p \%{buildroot}{\%{_unitdir},/opt/\%{name}}
pip3 install -r requirements.txt --no-index --find-links . --target=\%{buildroot}/opt/\%{name}/lib
unzip quality*.whl -d \%{buildroot}/opt/\%{name}
rm -rf \%{buildroot}/opt/\%{name}/quality-report-0.0.1.dist-info
cp config.yaml \%{buildroot}/opt/\%{name}
install -m 0755 \%{name}.service \%{buildroot}\%{_unitdir}/\%{name}.service

\%post
\%systemd_post \%{name}.service

\%preun
\%systemd_preun \%{name}.service

\%postun
\%systemd_postun \%{name}.service
rm -rf /opt/\%{name}

\%files
\%{_unitdir}/\%{name}.service
/opt/\%{name}/config.yaml
/opt/\%{name}/src/*
/opt/\%{name}/lib/*

\%changelog
```

This is a common RPM SPEC file, that performs the installation of the
application with a systemctl service. The most important part to comment is
the `install` section:
- `pip3` (as part of Python3.6 installation), installs the required
dependencies described in the `requirements.txt` file. This installation
happens in a `/lib` subdirectory of our application (`/opt/quality_report`)
  - This installation step happens here (and not in Windows) so that the proper
  repositories for RHEL can be used for download (since the app will run in
    RHEL distro)
  - The `/lib` directory is asssumed by the application as source of
  dependencies  because of the environment variable `PYTHONPATH`, defined in
  the systemctl service
- The application artifacts are properly unpacked in the `/opt/quality_report`
directory
- The app systemctl service is properly installed

## Put it all together: let's distribute!

### Create a distribution package

The following commands must be executed in the programmers's Windows 10 Box,
in the application directory root, using `Git Bash`:

```bash
# access yout virtualenv. It is assumed that all packages are already
# installed in your virtualenv - if not, pipenv install'em
$ pipenv shell

# execute the shell script that generated the dist package:
$ ./dist/generate-dist-package.sh
```

This will generate a `quality_report-dist-0.0.1.tar.gz` file for distribution
in the `dist\` directory. Copy this compacted file into a CentOS7 Virtual
Machine to continue.

### Generate a RPM installation package using a CentOS7 distro

In a `CentOS7 VM`, save the compacted file in some temporary
directory and run the following commands **as root**:

```bash
$ yum install dos2unix -y
$ tar zxvf ./quality_report-dist-0.0.1.tar.gz
$ cd dist
$ dos2unix ./generate-rpm-package.sh ./rpmbuild/SPECS/quality_report.spec
$ chmod +x ./generate-rpm-package.sh
$ ./generate-rpm-package.sh
```

This will generate a `quality_report-0.0.1-1.el7.x86_64.rpm` artifact in
the `rpmbuild/RPMS/x86_64` directory. Send it to your production/destination
box to perform the actual installation.

If, during the execution of `generate-rpm-package.sh` file, the following
errors start to show up:
`Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:877)'),)': /simple/certifi/`
, the most common cause is your Internet security program running on Windows.
{:.note}

### On the Production Box

Execute the following commands to perform the installation:

```bash
# remove ius community python36u (only if you are Usin CentOS and have both ius and epel)
$ sudo yum -y remove python36u python36u-devel python36u-pip \
    python36u-setuptools python36u-tools python36u-libs python36u-tkinter

$ sudo yum install -y quality_report-0.0.1-1.el7.x86_64.rpm
$ sudo systemctl start quality_report
```

The following commands can be run in order to validate the installation:

```bash
$ sudo systemctl status quality_report       ## must be active
$ curl -k https://quality_report:9091/       ## must return html data
$ ls /opt/quality_report                     ## must have config.yaml file,
                                             ##   lib and src dirs

## it must show application startup data
$ sudo cat /usr/lib/systemd/system/quality_report.service     
or
$ sudo cat /etc/systemd/system/quality_report.service
```

You can also check for the application documentation at any time:

```bash
$ sudo rpm -qi quality_report
```

And also to uninstall the application in a very straightforward way:

```bash
# the application service will be removed, along with /opt/quality_report dir
$ sudo yum remove -y quality_report
```

## Conclusion

This article aimed to provide a way to distribute a Python
[Flask](https://flask.palletsprojects.com/en/1.1.x/) app, developed in a
Windows workstation, to a RedHat Production Box without Internet access. In
order to perform this action, a intermediate CentOS7 box was required.
