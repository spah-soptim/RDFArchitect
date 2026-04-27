---
title: Troubleshooting
sidebar_position: 9
---

# Troubleshooting

The operational issues that come up most often, with concrete fixes.

## "Failed to fetch" / blank screen after login

**Symptom:** Frontend loads but the welcome page never shows datasets; browser console reports CORS or 502 errors.

**Likely causes:**

- The reverse proxy is not forwarding `/api` to the backend. Test with `curl https://your-host/api/datasets`.
- `frontend.url` on the backend doesn't match the actual public origin. CORS rejects the response.
- The session cookie is being dropped because `Secure` is `false` over HTTPS or `SameSite=strict` blocks the cross-site request.

**Fix:** Align `frontend.url`, your proxy routes, and the session cookie flags.

## Backend won't start: "Connection refused" to Fuseki

**Symptom:** Logs show the Spring context fails to initialise with a Jena `HttpException` against `:3030`.

**Fix:**

1. Check Fuseki is actually running: `curl http://fuseki:3030/$/ping`.
2. Check the URL: `DATABASE_HTTP_ENDPOINT` should match the in-cluster name, not `localhost`.
3. Check Fuseki credentials if your service account requires authentication.

## "Forbidden" when creating a dataset

**Symptom:** UI surfaces an error, backend logs show 403 from Fuseki.

**Fix:** The Fuseki service account RDFArchitect uses must have admin privileges (to create datasets). Update the Fuseki user database.

## Diagram is empty

**Symptom:** Package navigation shows classes, but the diagram canvas is blank.

**Likely causes:**

- A malformed multiplicity string in one of the attributes.
- A class missing a `cims:belongsToCategory` triple.

**Fix:** Browser console will log the offending IRI. Open the class in the class editor and fix the field. If you can't open it, examine the graph in Fuseki directly.

## SHACL view shows nothing

**Symptom:** SHACL view is empty for a class that should have generated shapes.

**Likely causes:**

- The class is unreachable from any `cims:Package`.
- The class's stereotype is `primitive` (primitive datatypes don't get full shape generation).

**Fix:** Verify the class membership and stereotype.

## Changelog grows without bound

**Symptom:** Fuseki disk use climbs steadily.

**Fix:** Prune old changelog entries:

1. Take a backup.
2. Use Fuseki's SPARQL endpoint to delete old `arch:changelog/...` resources beyond your retention window.
3. Run `tdb2.tdbcompact` to reclaim space.

## Snapshots fail to create

**Symptom:** "Snapshot failed" toast in the UI.

**Likely causes:**

- Fuseki write transaction conflict (multiple concurrent writers).
- Disk full.

**Fix:** Check Fuseki logs. Free disk; retry.

## Session keeps logging out

**Symptom:** User is bounced to the login screen periodically.

**Likely causes:**

- The proxy isn't propagating the session cookie correctly.
- Idle timeout at the proxy is shorter than the session.

**Fix:** Align `server.servlet.session.timeout` with the proxy's idle/refresh policy.

## Slow first render of a package

**Symptom:** First time a package is opened, layout takes seconds.

**Cause:** The diagram engine is computing a fresh layout. Subsequent renders are fast because the layout is persisted.

**Fix:** No fix needed; this is expected. If it's slow *every* time, check that the backend write transaction recording the layout actually committed (look for changelog entries with operation `layout-update`).

## "License file out of sync" CI error

**Symptom:** Backend or frontend CI fails on the license-file check.

**Fix:** Regenerate the licenses file (see [Dependencies](/developer-guide/dependencies)) and commit. This is a developer-side issue; it shouldn't surface in operations.

## Where to look first

For any operational issue:

1. **Backend logs.** Default location depends on your deployment; with the bundled Log4j2 config it's stdout.
2. **Fuseki logs.** Same — stdout in container, `logs/` directory in bare-metal Fuseki.
3. **Browser console.** Most UI issues surface there with specific error messages.
4. **Reverse proxy access log.** Tells you whether the request reached the backend at all.

## Asking for help

Open an issue at https://github.com/SOPTIM/RDFArchitect/issues with:

- Version (visible in the about dialog).
- Deployment topology (Compose? Kubernetes? Bare-metal?).
- Excerpts of relevant logs (redacted as needed).
- Reproduction steps.
