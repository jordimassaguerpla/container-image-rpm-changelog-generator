This is a simple command line tool used by the containment-rpm-docker to
dynamically create the changelog of a Docker image.

The tool requires three arguments to be provided:

  * `--new-packages` path to the `.packages` file just created by KIWI process.
  * `--old-packages` path to the `.packages` file created by the previous
    run of KIWI.
  * `--changelog` path to the file containing the old changelog of the Docker
    image.

The tool will parse the files create by KIWI, compare the old version with the
newer one and then produce a new changelog entry. If the older changelog is
provided the tool will add the new entry on top of the contents of the old file.

The resulting changelog is printed to the standard output.


## Dependencies

The tool is written in Ruby. The code has been written to be run also on older
versions of Ruby like the 1.8 version that is shipped with SLE11SP3.

No extra gem is required at runtime.

The comparison between the RPM versions is done by using a small python program
which takes advantage of the rpm binding.

We had to resort to this solution because the two gems providing bindings for
RPM do not work on SLE11SP3:

  * ruby-rpm-ffi: this one requires a recent version of librpm which is not
    available on SLE11SP3.
  * ruby-rpm: the package built for Ruby 1.8 has a runtime dependency issue
    (nothing provides rubygems); this is probably caused by the new packaging
    system we use to package gems into RPMs. The only solution is to use a more
    recent version of Ruby from openSUSE.org:devel:language:ruby. However this
    results in having a build system where there's no /usr/bin/ruby binary, just
    a /usr/bin/ruby<major.minor> one. This binary would evolve other the time,
    forcing us to change the contaiment script to point to the right version of
    it. Moreover relying on the latest version of Ruby from devel:languages:ruby
    doesn't seem to be a good choice; God knows what might break in the future.

The rpm-python package on the opposite is part of the official repositories both
on SLE11SP3 and on SLE12. Instead of rewriting the program in python Flavio
decided to take a shortcut and reply on the external python program.

The external program has been wrapped inside of the `RPM::Version` class which
maintains the same API of the class provided by ruby-rpm-ffi and ruby-rpm. This
would allow us to get rid of the python script once we kill SLE11SP3.
