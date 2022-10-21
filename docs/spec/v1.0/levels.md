# Security levels

<div class="subtitle">

SLSA is organized into a series of levels that provide increasing supply chain
security guarantees. This gives you confidence that software hasn’t been
tampered with and can be securely traced back to its source.

</div>

This page is an informative overview of the SLSA levels, describing their
purpose and guarantees. For the normative requirements at each level, see
[Requirements](requirements.md). For a background, see
[Terminology](terminology.md).

## What is SLSA?

SLSA is a set of incrementally adoptable security guidelines, established by
industry consensus. The standards set by SLSA are guiding principles for both
software producers and consumers: producers can follow the guidelines to make
their software more secure, and consumers can make decisions based on a software
package's security posture. SLSA's levels are designed to be incremental
and actionable, and to protect against specific classes of supply chain attacks.
The highest level in each track represents an ideal end state, and the lower
levels represent intermediate milestones with commensurate security guarantees.

Importantly, SLSA is intended to be a primitive in a broader determination of a
software's risk. SLSA measures specific aspects of supply chain security,
particularly those that can be fully automated; other aspects, such as developer
trust and code quality, are out of scope. Furthermore, each link in the software
supply chain has its own, independent SLSA level---in other words, it is not
transitive ([FAQ](../faq.md#q-why-is-slsa-not-transitive)). The benefit of this
approach is to break up the large supply chain security problem into tractable
subproblems that can be prioritized based on risk and tackled in parallel. But
this does mean that SLSA alone is not sufficient to determine if an artifact is
"safe".

> **TODO:** SLSA is in the eye of the beholder: software consumers make their
> own SLSA determinations, though in practice they may delegate to some
> authority.

## Who is SLSA for?

SLSA is intended to serve multiple populations:

-   **Project maintainers,** who are often small teams that know their build
    process and trust their teammates. Their primary goal is protection against
    compromise with as low overhead as possible. They may also benefit from
    easier maintenance and increased consumer confidence.

-   **Consumers,** who use a variety of software and do not know the maintainers
    or their build processes. Their primary goal is confidence that the software
    is authentic and has not been tampered with. They are concerned about rogue
    maintainers, compromised credentials, and compromised infrastructure.

-   **Organizations,** who are both producers and consumers of software. In
    addition to the goals above, organizations also want to broadly understand
    and reduce risk across the organization.

-   **Infrastructure providers,** such as build services and packaging
    ecosystems, who are critical to achieving SLSA. While not the primary
    beneficiary of SLSA, they may benefit from increased security and user
    trust.

## Levels and tracks

SLSA levels are split into *tracks*. Each track has its own set of levels that
measure a particular aspect of supply chain security. The purpose of tracks is
to recognize progress made in one aspect of security without blocking on an
unrelated aspect. Tracks also allow the SLSA spec to evolve: we can add more
tracks without invalidating previous levels.

| Track/Level | Requirements | Benefits |
| ----------- | ------------ | -------- |
| [Build L0]  | (none)       | (n/a)    |
| [Build L1]  | Attestation showing that the package was built as expected | Documentation, mistake prevention, inventorying |
| [Build L2]  | Signed attestation, generated by a hosted build service | Reduced attack surface, weak tamper protection |
| [Build L3]  | Hardened build service | Strong tamper protection |
| [Build L4]  | (not yet defined) | |
| [Source L…] | (not yet defined) | |

> Note: The [previous version] of the specification used a single unnamed track,
> SLSA 1–4, roughly corresponding to the equivalently numbered Build + Source
> levels.

## Build track

The SLSA build track describes the level of protection against tampering
during or after the build, and the trustworthiness of provenance metadata.
Higher SLSA build levels provide increased confidence that a package truly came
from the correct sources, without unauthorized modification or influence.

> **TODO:** Add a diagram visualizing the following.

Summary of the build track:

-   Set **project-specific expectations** for how the package should be built.
-   Generate a **provenance attestation** automatically during each build.
-   Build on **hardened services** that have been manually audited.
-   **Automatically verify** that each package's provenance meets expectations
    before allowing its publication and/or consumption.

Exactly how this is implemented is up to the packaging ecosystem (for open
source) or organization (for closed source), including: means of defining
expectations, what provenance format is accepted, whether reproducible builds
are used, how provenance is distributed, when verification happens, and what
happens on failure. Guidelines for implementers can be found in the
[requirements](requirements.md).

<section id="build-l0">

### Build L0: No guarantees

<dl class="as-table">
<dt>Summary<dd>

No requirements---L0 represents the lack of SLSA.

<dt>Intended for<dd>

Development or test builds of software that are built and run on the same
machine, such as unit tests.

<dt>Requirements<dd>

n/a

<dt>Benefits<dd>

n/a

</dl>
</section>
<section id="build-l1">

### Build L1: Provenance

<dl class="as-table">
<dt>Summary<dd>

Package has a provenance attestation showing that it was built as expected, but
the provenance is trivial to forge.

<dt>Intended for<dd>

Projects and organizations wanting to easily and quickly gain some benefits of
SLSA---other than tamper protection---without changing their build workflows.

<dt>Requirements<dd>

-   Up front, the package maintainer defines how the package is *expected* to be
    built, including the canonical source repository and build command.

-   On each build, the release process automatically generates and distributes a
    [provenance attestation] describing how the artifact was *actually* built,
    including: who built the package (person or system), what process/command
    was used, and what the input artifacts were.

-   Downstream tooling automatically verifies that the artifact's provenance
    exists and matches the expectation.

<dt>Benefits<dd>

-   Makes it easier for both maintainers and consumers to debug, patch, rebuild,
    and/or analyze the software by knowing its precise source version and build
    process.

-   Prevents mistakes during the release process, such as building from a client
    with local modifications.

-   Aids organizations in creating an inventory of software and build systems
    used across a variety of teams.

<dt>Notes<dd>

-   Provenance may be incomplete and/or unsigned at L1. Higher levels require
    more complete and trustworthy provenance.

</dl>

> **TODO:** Add "Scripted Build" if decided in
> [#498](https://github.com/slsa-framework/slsa/issues/498): "Build process is
> fully scripted/automated, with no manual steps."

</section>
<section id="build-l2">

### Build L2: Build service

<dl class="as-table">
<dt>Summary<dd>

Builds run on a hosted service that generates and signs the provenance, reducing
attack surface and increasing the difficulty to forge the provenance.

<dt>Intended for<dd>

Projects and organizations wanting to gain moderate security benefits of SLSA by
switching to a hosted build service, while waiting for changes to the build
service itself required by [Build L3].

<dt>Requirements<dd>

All of [Build L1], plus:

-   Builds run on a hosted build service that generates and signs the
    provenance itself.

-   Downstream verification of provenance includes authenticating the
    provenance.

<dt>Benefits<dd>

All of [Build L1], plus:

-   Prevents tampering after the build through digital signatures.

-   Moderately reduces the impact of compromised package upload credentials by
    requiring attacker to also exploit the build process.

-   Reduces attack surface by limiting builds to specific build services that
    can be audited and hardened.

-   Allows large-scale migration of teams to supported build services early
    while further hardening work ([Build L3]) is done in parallel.

</dl>
</section>
<section id="build-l3">

> **TODO:** If possible, avoid being overly specific about "signing".

### Build L3: Hardened builds

<dl class="as-table">
<dt>Summary<dd>

Builds run on a hardened build service that offers strong tamper protection. The
provenance is very difficult to exploit even for a determined adversary.

<dt>Intended for<dd>

Most software releases. Build L3 usually requires significant changes to
existing build services.

<dt>Requirements<dd>

All of [Build L2], plus:

-   Build service implements strong controls to:
    -   prevent runs from influencing one another, even within the same project.
    -   prevent secret material used to sign the provenance from being accessible to the user-defined build steps.

> **TODO:** Add requirement about survey and audit as per the v1.0 proposal.

<dt>Benefits<dd>

All of [Build L2], plus:

-   Prevents tampering during the build---by maintainers, compromised
    credentials, or other tenants.

-   Greatly reduces the impact of compromised package upload credentials by
    requiring attacker to perform a difficult exploit of the build process.

-   Provides strong confidence that the package was built from the official
    source and build process.

</dl>
</section>
<section id="build-l4">

### Build L4: (not yet defined)

Build L4 is not yet defined. For reference, [previous version] of SLSA defined a
"SLSA 4" that included the following requirements, which **may or may not** be
part of a future Build L4:

-   Pinned dependencies, which guarantee that each build runs on exactly the
    same set of inputs.
-   Hermetic builds, which guarantee that no extraneous dependencies are used.
-   All dependencies listed in the provenance, which enables downstream systems
    to recursively apply SLSA to dependencies.
-   Reproducible builds, which enable other systems to corroborate the
    provenance.

</section>
<section id="source-track">

## Source track (not yet defined)

The Source track is not yet defined but is expected to measure protection
against tampering of the source code. It will be defined in a future version of
the specification.

For reference, the [previous version] of SLSA included the following
requirements, which **may or may not** be part of a future Source track:

-   Strong authentication of author and reviewer identities, such as 2-factor
    authentication using a hardware security key, to resist account and
    credential compromise.
-   Retention of the source code to allow for after-the-fact inspection and
    future rebuilds.
-   Mandatory two-person review of all changes to the source to prevent a single
    compromised actor or account from introducing malicious changes.

</section>

<!-- Link definitions -->

[build l0]: #build-l0
[build l1]: #build-l1
[build l2]: #build-l2
[build l3]: #build-l3
[build l4]: #build-l4
[build]: #build-track
[previous version]: ../v0.1/levels.md
[provenance attestation]: terminology.md
[source l…]: #source-track