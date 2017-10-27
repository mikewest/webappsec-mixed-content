# Proposal: Mixed Content Level 2

Over the last few years, user agents have more or less converged on the behavior defined in our
[first pass at a Mixed Content spec](https://w3c.github.io/webappsec-mixed-content/). We have a
[robust test suite](https://github.com/w3c/web-platform-tests/tree/master/mixed-content), relatively
[good agreement on test results](https://wpt.fyi/mixed-content), and with luck we'll be able to push
that document through to REC in the relatively near-term (which is good, since it went to CR in
March 2015!). So, we have a solid foundation. What's next?

This document sketches a few problems we might want to solve together, and proposes solutions
browser vendors might be interested in implementing.

## Problems

### Optionally-blockable mixed content

Usage of optionally-blockable mixed content is still fairly pervasive (~2.4% of Chrome's page views,
the vast majority of which is image content); the current warnings and UX aren't having much effect
on driving the numbers down to something more negligible.

### User Interface

The current Mixed Content document suggests that user agents should degrade any security indicators
it might present if optionally-blockable mixed content is allowed to load. Chrome does this today by
dropping the green lock for pages like <https://mixed.badssl.com/>. Other vendors have similar
behavior: Safari and Edge both degrade to pretty much the same presentation as an HTTP page, while
Firefox keeps the lock, but badges it with a yellow warning triangle.

Chrome has an eventual goal of rendering all non-secure origins as [affirmatively
non-secure](https://www.chromium.org/Home/chromium-security/marking-http-as-non-secure), and
dropping the affirmative indication of security from HTTPS pages. Should we continue to distinguish
between loading optionally-blockable and blockable content in this future UX? Is there a user-facing
distinction between "Not secure because you loaded an image", and "Really not secure because you
instructed the user agent to load script"?

### User opt-ins

That last bit points to another UX question: the current document suggests that user agents may
offer users the ability to [override the restriction on blockable mixed
content](https://w3c.github.io/webappsec-mixed-content/#requirements-user-controls). Chrome
currently does so on desktop with a shield icon in the address bar on pages like
<https://mixed-script.badssl.com/>. Edge and Firefox have similar UX. Chrome on mobile devices and
Safari on all devices, on the other hand, do not offer this UX, and it's likely that we'll drop
it from Chrome on desktop in the future. Is it an option the spec needs to continue to provide?

### Migration Pains

Migrating from HTTP to HTTPS is still more painful than it ought to be due to the UX issues above,
and the spectre of hard-coded absolute URLs that will be blocked in a secure context.
`Upgrade-Insecure-Requests` is a partial solution, and has shown real value in the wild. But it
remains a fairly blunt opt-in mechanism that developers have to know about and choose to use. [HSTS
Priming](https://wicg.github.io/hsts-priming/) aimed to bump the defaults up a bit, but hasn't shown
enough value in based on the rumblings I've heard from Mozilla to be worth implementing.

## Proposals

1.  User agents should **upgrade rather than block** requests currently categorized as blockable
    mixed content. That is, when evaluating `<script src="http://example.test/js"></script>`, the
    user agent should rewrite the URL into `https://example.test/js` in the same way it would if
    `Upgrade-Insecure-Requests` was present. Doing this would give developers a better shot at
    transparently migrating from HTTP to HTTPS, even if they miss some critical `<script>` or
    `<iframe>` tags.
    
    (Credit where it's due: I'm pretty sure Tanvi and others suggested this several years ago. I've
    come to agree with her, despite my objections at the time.)


2.  User agents should **treat optionally-blockable mixed content as blockable by default** (and, if
    we accept the proposal above, that means that we'd upgrade it rather than blocking). Since,
    however, there are real use cases that still require loading optionally-blockable mixed content
    (image search on Bing, Google, and other search engines that have chosen not to proxy other
    folks' images for whatever reason; podcast applications like <https://overcast.fm/>; etc.), we
    could could allow developers to opt-into status quo behavior in some way (perhaps a
    `Mixed-Content: I-ve-Got-Some-Oh-Noes` header?) at the cost of degrading their security
    indicator in some way. The assumption underlying this proposal is that a large fraction of
    optionally-blockable mixed content is loaded accidentally or is not critical to the site's
    functionality. If this assumption holds, then requiring site owners to explicitly opt in to
    optionally-blockable mixed content will dramatically reduce the amount of optionally-blockable
    mixed content that users end up loading.

3.  User agents should **deprecate and remove `Upgrade-Insecure-Requests`** to reduce the platform's
    overall complexity, as the two proposals above would seem to obviate it entirely.

4.  User agents should **remove their shield UX more broadly**, perhaps following Safari's lead by
    removing it entirely, perhaps compromising based on use cases by moving it out of the address
    bar into developer tooling or extensions.
