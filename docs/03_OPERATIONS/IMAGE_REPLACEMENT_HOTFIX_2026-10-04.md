# Existing activity image replacement

Status: Partial. Implementation, automated validation and preview: Done. Authenticated production replacement: Planned (manual smoke). Size: L, contained hotfix (authenticated editor, Storage upload and save lifecycle).

## Branch context

- Active implementation branch: `main`, starting at `b9d15ed`.
- Current maintenance code and docs are authoritative. Existing user attachments stay outside commits.

## Current state

- `/internal/activities/:activityId` renders `InternalApprovedActivityPage` and the shared `ScoutDraftReviewForm`, but omits its image-upload props.
- The existing uploader, validation/optimization and `uploadDraftCoverImage` service already support safe image uploads to the activity's linked draft. Existing payload mappers and approved-save RPC preserve `image_url`.
- `/internal/drafts/new` already has local file selection and a preview. Editable `/internal/drafts/:draftId` already uploads images. Approved drafts link to the affected activity editor.
- Profile publication/correction routes mentioned in the supplied reference are retired in the maintenance branch; their retained form already enables upload. They will not be reactivated.
- The approved activity editor has no local recovery today. Existing creation/recovery helpers must remain unchanged.

## Goal and scope

Enable the shared human image selector for existing editable activities, replacing either a placeholder or an existing image. Preview the chosen file immediately, upload on save and persist the returned path through the existing approved-save service. Preserve the current image when no replacement is selected. Keep publication status and all other field contracts unchanged.

## Touched files

- `src/pages/InternalApprovedActivityPage.jsx`
- `src/pages/InternalApprovedActivityPage.css` (localized containment after mobile verification found the new selector clipped)
- `scripts/build-public-catalog-backup.mjs` (verified PNG mislabelled as JPEG blocks the required public build)
- `scripts/public-catalog-backup-check.mjs` (regression and rejection cases for that precise format mismatch)
- `docs/03_OPERATIONS/IMAGE_REPLACEMENT_HOTFIX_2026-10-04.md`

## Out of scope

Shared form design, upload service, schema/RLS/RPC/Storage policies, routes, user/profile/publisher access, price, contacts, center, age, schedule, catalog design, billing and unrelated recovery behavior. No deletion, metadata mutation or overwrite of existing image objects. No broad mobile redesign or general MIME relaxation.

## Risks

- Saving or toggling visibility while an image upload is still pending.
- Losing a replacement through an unsaved publish/unpublish action.
- Upload succeeding but save failing, causing unnecessary reuploads on retry.
- A stale asynchronous response changing another activity after navigation.
- Accidental persistence of File/blob/base64 content.

## Assumptions avoided

- Keep JPG/PNG/WebP, 5 MB limit and existing optimization; do not introduce SVG uploads or a new upload implementation.
- Use the linked draft ID for the existing upload service. Confirmed persistence comes from the existing reload RPC.
- Only a stored image path enters the payload. Local File and preview object URL remain in memory and are released.
- Frontend fixtures do not establish an authenticated production save. Any remaining live smoke must be reported.

## Implementation sequence

1. Add selected-file state and managed preview to the existing approved activity page; enable the shared selector and hide the technical reference on this surface.
2. Upload a selected replacement before the existing save, preserve a successful upload for save retries, and serialize image/save/publication writes.
3. Guard stale responses on route changes and keep unsaved image changes from being discarded by publication toggles.
4. Run independent read-only review and fix only identified issues.
5. Verify placeholder/real replacement, unchanged images, invalid files, upload/save failures, retries, navigation and existing creation/recovery with isolated browser fixtures. Run repository checks and diff checks.
6. Save one focused commit after preview verification; publish under the user's existing direct authorization and verify production. Record authenticated-write limitations separately.

## Validation

- `npm.cmd run check` (includes production build) and diff/cached diff checks.
- Real editor in an isolated browser fixture with actual form, upload optimizer, mappers and services; only Supabase responses/storage are simulated. No production activities are changed for tests.
- Mobile screenshot, current preview, save/reopen, publication status and image path in public/detail presentation.
- Creation uploader and its safe recovery behavior remain operational. Independent review checks write ordering, file cleanup and error paths.

## Pending after closure

Record executed evidence and any external smoke that could not be performed. Stop before any new Storage/RLS/RPC changes; none are currently required.

## Findings and contained fix pack

- Independent editor review: PASS, no blocking code findings.
- Browser fixtures: 18 functional/regression cases pass, but the approved editor overflows a 390px viewport to 512px and clips the selector. Add only grid/column containment in this page's CSS before repeating mobile checks.
- Repository checks pass through all audits and Vite compilation; the real backup export fails. Read-only investigation confirms an existing public image for `Escoltem els cetacis del Garraf` is served as `image/jpeg` with a PNG byte signature. Views remain readable (11 activities and 15 contacts).
- Expand the exact touched scope before edits: permit only a verified PNG signature with either PNG or JPEG image MIME to produce a `.png` backup file. Keep unsupported MIME, non-raster content, arbitrary SVG, DNS/SSRF, size and completeness rejection unchanged. No production image or metadata changes are needed.
- Add regression assertions for this mismatch and rejection cases, then repeat backup/build and mobile checks. These are concrete verification findings required to deliver the hotfix, not additional product features.

## Executed validation

- Final `npm.cmd run check`: PASS, including static/contracts, contact messages, publisher SQL, backup, maintenance permissions/center lifecycle and production build. Initial backup failure was diagnosed and resolved by the contained PNG compatibility fix above.
- Public backup now exports 11 activities, 15 contacts and 11 copied images. A verified PNG served with JPEG metadata is copied with a `.png` extension. Unsupported MIME and non-raster/SVG/script content still fail the regression checks.
- Independent read-only review of the editor and contained fix pack: PASS, no blocking findings.
- Isolated browser fixture: 20 scenarios PASS with actual pages, shared form, optimizer, mappers, services and catalog. Only Supabase RPCs and Storage are simulated. Covers placeholder/real replacement, all accepted formats, unchanged images, rejected SVG/oversize files, upload and save failures, save retry without duplicate upload, write locking/double clicks, stale responses after navigation, save/reopen, republish and public card/detail image presentation.
- Existing creation upload/save and safe recovery, pending-draft uploader and approved-draft read-only handoff: PASS. Recovery retains sanitized fields and a reselect-file warning; no File/blob/base64/auth identity is stored.
- Mobile uses real scoped CSS in a fresh session: viewport/document 390/390, complete selector and preview, scrollable Markdown toolbar and reachable save control. Desktop/mobile screenshots inspected. No runtime errors or external requests in the fixture. Fixture server and browsers closed.
- Temporary evidence: `nensgo-image-edit-fixture-2026-10-04/results.json`, `verify-browser.mjs`, `image-selection-mobile.png`, `replacement-saved-mobile.png`, and desktop screenshots in the host temporary directory. No fixture is committed or deployed.

## Preview and release

- Preview: `dpl_2X58HU4b8ikKTeUVtFC8T8pGKAQo`, `https://mvp-pevjmteee-dibrandons-projects.vercel.app`, READY. Source is the staged Git tree `721af4940a978d0d2d17ac01d4a2c8f726f6d5b0`; all runtime files are retained exactly in the final focused commit. Only validation documentation changes afterward.
- Protected preview HTTP: PASS for the matching entry assets, exact approved editor JS/CSS, catalog/manifest and all copied images. The formerly blocked PNG returns PNG content type and signature from the backup. Retired statistics API 410 and unauthenticated backup API 401 remain intact.
- Preview access uses official Vercel protection; no protection setting is disabled and no bypass/session secret is committed.
- Preview browser: 5 scenarios PASS. Anonymous editor access redirects to `/usuario` with no uploader exposed. Public home renders 11 activities. Simulated database failure uses the actual published generation with 15 contacts, 11 images and 5 free labels. The copied cetacean PNG decodes at 1000×530 in card/detail; mobile stays 390/390 and renders its two-contact selector. No unexpected errors or external actions.
- Remote evidence and reusable script: `nensgo-image-edit-preview-2026-10-04/preview-browser-results.json`, `verify-remote.mjs` and mobile screenshots in the host temporary directory. The script can verify production without a preview session.
- Production deployment and Git synchronization will be confirmed in the final handoff after this single focused commit. A real authenticated production image replacement remains a separate smoke; tests do not modify a live administrator's activity.

Administrator smoke after release: open an existing activity from the private activity list, select an image, save changes, reopen to confirm it, then republish if appropriate and create the public backup. No technical image URL is required.
