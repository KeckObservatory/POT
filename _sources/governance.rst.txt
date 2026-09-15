Governance
==========

This page describes the governance policy for mKTL. Any individual or
organization contributing to mKTL, regardless of their role, implicitly
agrees to this policy.


Project support
---------------

The principal customer for mKTL and mKTL-based systems is
`W. M. Keck Observatory <https://keckobservatory.org/>`_ (WMKO). As such,
WMKO is committed to the ongoing development and maintenance of mKTL, and
will support the participation of at least one maintainer for the duration
of mKTL's operational use, and will internally support contributors as needed
for ongoing development and operational fixes.

Other participating institutions shall make a similar commitment commensurate
with their needs.


Approval
--------

Changes to mKTL governance require a majority of all maintainers, and ideally
a consensus from participating contributors. Significant changes may also
require the approval from supporting institutions.

Approval of any other changes requires:

  #. A minimum of two maintainers from WMKO
  #. A minimum of one maintainer from outside WMKO

There can be overlap between these roles, for example, if a maintainer is
from an external institution. Contributors are not expected to approve their
own contributions, except in the case where no other approvals are available,
for example if there is only one maintainer and their contribution requires
approval. Organization administrators may be appealed to bypass the normal
approval process when there are unacceptable delays or an inadequate number
of maintainers are available. Bypassing the normal approval process must be
justified and this authority should be used sparingly, if at all.


Roles
-----

User
^^^^

A user is anyone that uses mKTL for any task. Users are encouraged to
provide feedback on any and all topics, especially with regard to any
strengths or weaknesses of mKTL from their perspective.

Contributor
^^^^^^^^^^^

Contributors are a subset of users.

A contributor is anyone who manipulates the source code repository in
any context: submitting issues, commenting on issues, submitting pull
requests, or any similar activity is considered a contributor. There
are no formal expectations or commitments associated with being a
contributor, nor are there any limits on how much time a contributor
might provide to the project.

Contributor status does not confer approval authority, which rests solely
with maintainers.

All contributors implicitly agree to abide by the mKTL code of conduct.

Maintainer
^^^^^^^^^^

Maintainers are a subset of contributors.

A maintainer is an individual with commit and merge access on the mKTL
repository. There will always be an odd number of designated maintainers
to ensure a majority vote is possible, should voting situations arise.
The minimum number of maintainers is one.

Maintainers are expected to make routine contributions to mKTL, in the
form of reviewing and merging pull requests, actively participating in
discussions, and building consensus around any decisions. On an annual
basis this is expected to be a minimum of a 40-80 hour time commitment
from each maintainer.

Any changes to the list of maintainers must be approved by a majority
of the current maintainers. The initial set of maintainers is expected
to include representatives from W. M. Keck Observatory, University
of California Observatories, and the California Institute of Technology.

All maintainers implicitly agree to abide by the mKTL code of conduct.


Communication
-------------

mKTL is maintained as a GitHub project. Issue tracking will occur
via GitHub; discussion of pull requests should primarily occur in the
discussion thread associated with the pull request. Secondary discussions
will occur in the #mktl Slack channel in the WMKO Software Coordination
workspace. Side conversations are inevitable, but should be brought back to
a common space where all contributors can participate in the discussion.


Contributed code
----------------

Pull requests
^^^^^^^^^^^^^

All code contributions will occur via pull request. Pull requests shall
be targeted and concise, ideally addressing a single topic (whether or not
it is separately filed as an issue), and being manageable to review. Pull
requests should take less than an hour to review; similarly, the number of
lines changed should be kept low, with 50-200 lines being a reasonable target,
and 500 lines as an upper bound. These are not firm requirements, but staying
within these guidelines helps reduce the burden on maintainers and other
reviewers, and increases the likelihood that a pull request can be properly
assessed and quickly merged. This assessment process is expected to occur
within a week of submission.

Informative commit messages and other inline documentation shall be
leveraged whenever practical. Clear expressions of intent go a long way
towards understanding code, both for reviews and future debugging.

All pull requests must pass the unit test suite with no errors.

Code style
^^^^^^^^^^

Adherence to
`PEP-8 <https://peps.python.org/pep-0008/>`_ is strongly suggested but
is not rigidly enforced; absent a formal code style document
the established practices in the mKTL code shall be followed.
This includes handling of whitespace, quoting, capitalization practices,
variable naming, type hints (or the absence thereof), and the use of the
English language.

Compatibility
^^^^^^^^^^^^^

Backwards compatibility is a priority for mKTL and all contributions shall
take care to adhere to the established minimum package versions.
Requiring a newer release of Python, for example, shall be
a major topic of discussion among maintainers and contributors; use of
incompatible language features or module functionality will result in
rejection of a pull request.

Copyright
^^^^^^^^^

All contributors relinquish individual copyright claims for any contributed
code. Code subject to an external copyright shall not be submitted in any
form.


Releases
--------

Periodic stable releases will be tagged from the main repository; this should
be the resource of choice for any users requiring stable behavior. The main
branch makes no promises of stability, other than it is expected to be self
consistent and functional. Branches are intended for specific feature or
issue related development, and will not have any meaningful persistence.

mKTL will follow the Python versioning scheme described in
`PEP-440 <https://peps.python.org/pep-0440/>`_, though perhaps with more
clarity in the `Python packaging documentation <https://packaging.python.org/en/latest/discussions/versioning/>`_
as `semantic versioning <https://semver.org/>`_, with a three part
version number, concatenated with a '.' character (major.minor.patch):

  * major, for changes breaking backwards compatibility
  * minor, for backwards-compatible additions or changes
  * patch, for backwards-compatible bug fixes

A description of all changes for a release shall be summarized in a changelog.


User support
------------

There is no dedicated system to provide mKTL support to end users; at the
present time, any potential users of the system have direct contact information
for one or more maintainers; when such questions arise, contributions should
be made to address any deficiencies in the online documentation.

The existence of project support does not imply there is a warranty or other
resources that the user base is entitled to draw upon should problems arise.
