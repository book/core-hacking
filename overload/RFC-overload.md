# Support overloaded objects in join(), substr() builtins

## Preamble

    Authors: Eric Herman <eric@freesa.org>, Philippe Bruhat <book@cpan.org>
    Sponsor: Paul Evans <leonerd@leonerd.org.uk> 
    ID:      
    Status:  Draft

## Abstract

Today, `overload` is incomplete and surprising for string operations:

* the `join` builtin should be using the `concat` (`.`) overload
  when operating on overloaded objects (including the separator),
* `overload` should support the `substr` operation.

## Motivation

The perl builtin `join`, when operating on overloaded objects will
stringify them and the result will no longer be an object of the class
with overloads, even if the `concat` (`.`) function is overloaded.

Additionally, the `overload` module does not currently support overloading
the `substr` operation.

The current behaviour is inconsistent and confusing.

There is at least one CPAN module which works around the join
deficiency, (the [`join` function of
`String::Tagged`](https://metacpan.org/pod/String::Tagged#join), but not
universally.

The CPAN module
`[overload::substr](https://metacpan.org/pod/overload::substr)`) is a
complete solution, but we think that deferring to CPAN when Perl should
do the right thing is not a satisfying answer.

## Rationale

Extending `overload` to support `substr`, and `join` to use `concat`
would be less surprising and more consistent with the "Do What I Mean"
spirit of Perl.

Adding this to Perl would make `overload::substr` obsolete and simplify
the implementation of `String::Tagged` and other similar modules which
have written their own `join`.

## Specification

The documentation for `join` would be amended with the following:

> If any of the arguments (`EXPR` or `LIST` contents) are objects which
> overload the concat (`.`) operation, then that overloaded operation
> will be invoked.

The documentation for `overload` would be amended to extend the complete
list of keys that can be specified in the `use overload` statement to
include `substr`. The module `overload::substr` code and documentation
should be used as a reference and guide for implementation in core. Note
that the current documentation for this module highlights a need for
additional tests, which must be a part of a core implementation.

An open question is whether `split` requires an overload target,
and should a fallback implementation be provided using the
overloaded `substr`?

## Backwards Compatibility

If code relies upon the `join` function to convert objects to plain
strings, this would break that code, thus a feature flag seems
necessary.

There is no need autogenerate an implementation for `substr` if it is
missing, as the stringification will happen as it does now.

Modules which implement their own `join` would need to add
version-conditional logic to work with different versions of Perl.

`overload::substr` would become an optional dependency for modules
that needs to support older Perls.

## Security Implications

We do not yet foresee security issues. Guidance is welcome.

## Examples

    ```perl
    # this reduce works as expected with an overloaded concat
    my $ret = reduce { ( $a . $sep ) . $b } @list;

    # this join() surprisingly uses stringification of $sep,
    # even when concat is overloaded
    my $ret = join $sep, @list;
    ```
===== WE LEFT OFF HERE =====

PEPs have this as "How to Teach This". That's a valid goal, but there are different audiences (from newcomers, to experienced Perl programmers unfamiliar with your plan).

Most of us are not experienced teachers, but many folks reading your RFC are experienced programmers. So probably the best way to demonstrate the benefits of your proposal is to take some existing code in the core or on CPAN (and not your own code) and show how using your new feature can improve it (easier to read, less buggy, etc)

## Prototype Implementation

Is there something that shows the idea is feasible, and lets other people
play with it? Such as

* A module on CPAN
* A source filter
* Hack the core C code - fails tests, but lets folks play

## Future Scope

Were there any ideas or variations considered that seemed reasonable follow-ons from the initial suggestions, that we don't need to do now, but want to revisit in the future? If there are parts of the design marked as "intended for future expansion", then give an idea here of what direction is planned.

If that future expansion relies on extending the syntax, then ensure that the first implementation has tests to assert that the extended syntax remains a syntax error.

## Rejected Ideas

Why this solution/this syntax was better than the obvious alternatives.

We've seen before that there will be **F**requently **A**sked **Q**uestions.
eg *Why not have different behaviour in void context?*

Likely the answer is in the previous discussion **somewhere**, but most people won't stop to read all the comments. It needs to be easy to find, and updated as it becomes clear which questions are common.

Hence it **needs** to be in the RFC itself. Without this, the RFC process as a whole won't scale.

## Open Issues

Use this to summarise any points that are still to be resolved.

## Copyright

Copyright (C) 2038, A.U. Thor.

This document and code and documentation within it may be used, redistributed and/or modified under the same terms as Perl itself.
