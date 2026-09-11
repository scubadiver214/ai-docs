# Menu Image Blob Storage Locations

This document records the current Menu Manager, Menu Publisher, and published-menu image storage behavior for menu images. It is intended to support future automation for image upload, validation, and processing.

Evidence was collected from backend code, frontend upload code, PostgreSQL metadata and data, and direct read-only Azure Blob Storage inspection. Secret values are intentionally excluded. Connection strings, account keys, SAS tokens, API keys, passwords, and bearer tokens should not be added to this document.

## Evidence Status

| Evidence type | Status |
| --- | --- |
| Backend code paths | Reviewed in [MenuCreationDataController.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Api/Controllers/MenuCreationDataController.cs&_a=contents), [ImageUploadService.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Patch/ImageUploadService.cs&_a=contents), [BlobService.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Patch/BlobService.cs&_a=contents), [ImageDownloadService.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Patch/ImageDownloadService.cs&_a=contents), and [MenuCreationDataService.Image.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/MenuCreationServices/MenuCreationDataService.Image.cs&_a=contents). |
| Frontend upload path | Reviewed in [useImageUploader.ts](https://lce-digital.visualstudio.com/CVtoCloud/_git/Caesar-Vision-Menu-Manager?path=/src/api/dictionary/hooks/useImageUploader/useImageUploader.ts&_a=contents) and [menuItemCreation.ts](https://lce-digital.visualstudio.com/CVtoCloud/_git/Caesar-Vision-Menu-Manager?path=/src/types/menuItemCreation/menuItemCreation.ts&_a=contents). |
| PostgreSQL | Read-only queries were run against localhost, Dev, Test, and Prod `master-menu` profiles. |
| Azure Blob Storage | Read-only Azure Storage commands were run using the signed-in Azure extension context `bill.rodenbaugh@lcecorp.com` in tenant `LCE Corp`. Some accounts/containers were accessible, while DMB, Deliverect, and work-container blob inventory returned `AuthorizationPermissionMismatch` with the current RBAC. |

## High-Level Flow

1. The frontend posts multipart form data to `/MenuCreation/ImageUpdateView` with optional `imageFile` and JSON `imageDetail`.
2. `MenuCreationDataController.ImageUpdateView` calls `ImageUploadService.ProcessImageUploadAsync`.
3. `ImageUploadService` validates that `ViewId` is the shared Images view (`6`) and that `DeploymentSettings.EnvironmentId` is a supported deployment environment.
4. `MenuCreationDataService.GetImageUploadViewAsync` calls PostgreSQL routine `api_menu_create_get_view_image_upload`.
5. PostgreSQL returns `ImageUploadViewDTO` rows that include work URL, final URL, path extras, image type, image size, file extension, save-as filename, and `IsUXVisible`.
6. `BlobService.Upload` creates a `BlobContainerClient` from `UrlBaseWork` or `UrlBase`, then writes blob name `/{path}{fileName}` using `DefaultAzureCredential`.
7. `ImageUpdateView` writes initial edits to `UrlBaseWork`. `SaveImage` accepts an `ImageUploadViewDTO` and writes to `UrlBase` plus `UrlBaseExtra` for final/published destinations.
8. Only rows marked `IsUXVisible` update visible image metadata with `api_menu_create_update_view_image`.

`AzureBlobStorage` settings in appsettings are not the source of truth for menu-image destinations. Current evidence indicates that config section is for CSV/importer storage and should not be used to infer menu image paths.

## Business Rules Inferred From Code And Data

- Image storage is environment-aware. `DeploymentSettings.EnvironmentId` selects the deployment URL rows returned by SQL: `1 = Production`, `2 = QA/Test`, `3 = Development`.
- Image storage is organization-aware. Final destination rows repeat by root organization; automation must resolve organization context before choosing a destination.
- Image storage is service-location/channel-aware. SQL maps instore rows to Store destinations, online rows to both Online Web and Online Mobile, DMB-capable instore language rows to DMB, and other service-location cases to Deliverect.
- Image storage is language-aware. `url_add_language` and `url_add_language_after_extra` control whether culture code appears before or after extra path segments.
- Work upload storage and final/published storage are separate layers. Work uploads use `system_attribute.MenuCreationImageWorkeUrl`; final destinations use `menu_create_deployment_root_organization_image`; published-menu root URL columns live in `organization_as_menu_root`.
- The DB function output is the automation source of truth for `UrlBaseWork`, `UrlBaseWorkExtra`, `UrlBase`, `UrlBaseExtra`, filename, image type, image extension, image size, and visibility.
- API-driven upload through `/MenuCreation/ImageUpdateView` is the safest automation contract because it preserves SQL-derived metadata, validation, audit behavior, and post-upload metadata updates.
- Direct blob upload without the API must still retrieve the same DB metadata and update image metadata through supported routines. Direct database writes should be treated as unsafe.
- Blob objects are written by `BlobService` as `/{path}{fileName}`. Automation that bypasses the API must account for the leading slash behavior.
- Uploads use `ContentType = application/octet-stream` and `CacheControl = no-store, no-cache, must-revalidate` in code. Azure-verified blobs also show `application/octet-stream` for current menu-image uploads.
- If `ImageFileNameSaveAsAddExt` is false, the upload file extension must match the extension already in `ImageFileNameSaveAs`; otherwise the API throws.
- If `ImageFileNameSaveAsAddExt` is true, the upload extension is appended to `ImageFileNameSaveAs`.
- If no save-as name exists, SQL uses `image_file_uuid` and expects extension append.
- Alt-text-only updates require an existing image filename. Updating alt text without an existing image is rejected.
- Non-UX-visible rows may participate in blob upload fan-out but do not drive the visible image metadata update.
- One logical image change can fan out to multiple channel/language rows.
- Store images are PNG with expected sizes `709x312`, `325x360`, and `342x166`.
- Online Web images are JPG with image types `Web_ItemDetailImage` and `Web_MenuImage`.
- Online Mobile images are JPG with image types `Mobile_ItemDetailImage` and `Mobile_MenuImage`.
- Deliverect images are JPG with image type `full`.
- DMB images are PNG with image types `DMB_Normal` and `DMB_SodiumWarning`, and expected sizes `610x400` and `384x469`.
- Environment data can contain cross-environment references. Dev final online destinations currently point at `testcumobileimagesasa`. Prod `organization_as_menu_root` has org 8 Deliverect pointing at `https://testdeliverectimages.littlecaesars.com/read-bucket//`. Treat those as data findings to verify before automating.

## Azure Storage Account Inventory

Azure Resource Graph was queried across all accessible subscriptions. The default subscription was `lce-sandbox-dev`, but the menu image accounts live in development, testing, and production subscriptions.

| Environment | Account | Subscription | Resource group | Role in image flow | Public blob access |
| --- | --- | --- | --- | --- | --- |
| Dev | `devcumenucreationimages` | `lce-development` | `lce-development-centralus-menumgr` | Work/menu-creation images | true |
| Dev | `devcuassetsasa` | `lce-development` | `lce-development-centralus-cvcloud-assets` | Store/static website images | false |
| Dev | `testcumobileimagesasa` | `lce-testing` / referenced by Dev DB | `lce-testing-centralus-onlo-mobileimages-rg` | Dev/Test online images | true |
| Dev | `devcudeliverectimages` | `lce-development` | `lce-development-centralus-cvcloud-menupublisher-rg` | Deliverect images | false |
| Dev | `devcudmbimages` | `lce-development` | `lce-development-centralus-cvcloud-digitalmenuboard-rg` | DMB images | true |
| Test/QA | `testcumenucreationimages` | `lce-testing` | `lce-testing-centralus-menumgr` | Work/menu-creation images | true |
| Test/QA | `testcuassetsasa` | `lce-testing` | `lce-testing-centralus-cvcloud-assets` | Store/static website images | false |
| Test/QA | `testcumobileimagesasa` | `lce-testing` | `lce-testing-centralus-onlo-mobileimages-rg` | Online images | true |
| Test/QA | `testcudeliverectimages` | `lce-testing` | `lce-testing-centralus-cvcloud-menupublisher` | Deliverect images | true |
| Test/QA | `testcudmbimages` | `lce-testing` | `lce-testing-centralus-cvcloud-digitalmenuboard-rg` | DMB images | true |
| Prod | `prodcumenucreationimages` | `lce-production` | `lce-production-centralus-menumgr` | Work/menu-creation images | true |
| Prod | `prodcuassetsasa` | `lce-production` | `lce-production-centralus-cvcloud-assets` | Store/static website images | true |
| Prod | `prodcumobileimagesasa` | `lce-production` | `lce-production-centralus-onlo-mobileimages-rg` | Online images | true |
| Prod | `prodcudeliverectimages` | `lce-production` | `lce-production-centralus-cvcloud-menupublisher` | Deliverect images | true |
| Prod | `prodcudmbimages` | `lce-production` | `lce-production-centralus-cvcloud-digitalmenuboard-rg` | DMB images | true |

## Database-Driven URL Sources

### Work Upload Base

`system_attribute.attribute_name = 'MenuCreationImageWorkeUrl'` supplies `UrlBaseWork`. The spelling `Worke` is current database behavior.

| Environment | DB value | Azure verification |
| --- | --- | --- |
| localhost | `https://testcumenucreationimages.littlecaesars.com/menu-creation` | Local data only; not environment truth. |
| Dev | `https://devcumenucreationimages.littlecaesars.com/menu-creation` | Account and `menu-creation` container exist. Blob inventory denied by RBAC. |
| Test/QA | `https://testcumenucreationimages.littlecaesars.com/menu-creation` | Account and `menu-creation` container exist. Blob inventory denied by RBAC. |
| Prod | `https://prodcumenucreationimages.littlecaesars.com/menu-creation` | Account and `menu-creation` container exist. Blob inventory denied by RBAC. |

SQL normalizes the work base by adding a trailing slash and appending `menu-creation/` if missing. Work path extra is derived from `url_name` as lowercase, spaces replaced with hyphens, then a trailing slash: `Store -> store/`, `Online Web -> online-web/`, `Online Mobile -> online-mobile/`, `Deliverect -> deliverect/`, `DMB -> dmb/`. SQL then appends the language culture code.

Example work-upload URL shapes:

| Environment | Pattern | Example URL shape | Verification |
| --- | --- | --- | --- |
| Dev | Online Web work upload | `https://devcumenucreationimages.littlecaesars.com/menu-creation/online-web/en-us/{blobName}` | Container exists; blob inventory denied. |
| Test/QA | Store work upload | `https://testcumenucreationimages.littlecaesars.com/menu-creation/store/en-us/{blobName}` | Container exists; blob inventory denied. |
| Prod | DMB work upload | `https://prodcumenucreationimages.littlecaesars.com/menu-creation/dmb/en-us/{blobName}` | Container exists; blob inventory denied. |

The culture in these examples is illustrative. Actual culture comes from `api_menu_create_get_applicable_languages` through `api_menu_create_get_view_image_raw`.

### Final/Published Deployment Destinations

`menu_create_deployment_root_organization_image` supplies final image destinations.

| Environment | Target | Account/container | Extra path | Language rule | Image types | Extensions/sizes |
| --- | --- | --- | --- | --- | --- | --- |
| Dev | Store | `devcuassetsasa` / `$web` | `1/menu/` | language before extra | `full`, `half`, `quarter` | PNG, `709x312`, `325x360`, `342x166` |
| Dev | Online Web | `testcumobileimagesasa` / `new-menu` | `web/` | language after extra | `Web_ItemDetailImage`, `Web_MenuImage` | JPG |
| Dev | Online Mobile | `testcumobileimagesasa` / `new-menu` | `mobile/` | language after extra | `Mobile_ItemDetailImage`, `Mobile_MenuImage` | JPG |
| Dev | Deliverect | `devcudeliverectimages` / `read-bucket` | none | language before filename | `full` | JPG |
| Dev | DMB | `devcudmbimages` / `read-bucket` | none | no language | `DMB_Normal`, `DMB_SodiumWarning` | PNG, `610x400`, `384x469` |
| Test/QA | Store | `testcuassetsasa` / `$web` | `1/menu/` | language before extra | `full`, `half`, `quarter` | PNG, `709x312`, `325x360`, `342x166` |
| Test/QA | Online Web | `testcumobileimagesasa` / `new-menu` | `web/` | language after extra | `Web_ItemDetailImage`, `Web_MenuImage` | JPG |
| Test/QA | Online Mobile | `testcumobileimagesasa` / `new-menu` | `mobile/` | language after extra | `Mobile_ItemDetailImage`, `Mobile_MenuImage` | JPG |
| Test/QA | Deliverect | `testcudeliverectimages` / `read-bucket` | none | language before filename | `full` | JPG |
| Test/QA | DMB | `testcudmbimages` / `read-bucket` | none | no language | `DMB_Normal`, `DMB_SodiumWarning` | PNG, `610x400`, `384x469` |
| Prod | Store | `prodcuassetsasa` / `$web` | `1/menu/` | language before extra | `full`, `half`, `quarter` | PNG, `709x312`, `325x360`, `342x166` |
| Prod | Online Web | `prodcumobileimagesasa` / `new-menu` | `web/` | language after extra | `Web_ItemDetailImage`, `Web_MenuImage` | JPG |
| Prod | Online Mobile | `prodcumobileimagesasa` / `new-menu` | `mobile/` | language after extra | `Mobile_ItemDetailImage`, `Mobile_MenuImage` | JPG |
| Prod | Deliverect | `prodcudeliverectimages` / `read-bucket` | none | language before filename | `full` | JPG |
| Prod | DMB | `prodcudmbimages` / `read-bucket` | none | no language | `DMB_Normal`, `DMB_SodiumWarning` | PNG, `610x400`, `384x469` |

## Azure-Verified Concrete Examples

These examples were confirmed with direct read-only Azure Blob Storage calls. Metadata shown is from the tool result.

| Pattern | Environment | Source data | Account/container | Blob name | Full URL | Last modified | Bytes | Content type |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Store | Test/QA | `corp_menu_item_availability_image`, item `1000081`, `full` | `testcuassetsasa` / `$web` | `en-us/1/menu/bb8339fab82d4db79fbb8c3e34555b4c.png` | `https://testcuassetsasa.blob.core.windows.net/$web/en-us/1/menu/bb8339fab82d4db79fbb8c3e34555b4c.png` | `2026-06-19T14:44:56Z` | `2257` | `application/octet-stream` |
| Store | Test/QA | `corp_menu_item_availability_image`, item `1000057`, `half` | `testcuassetsasa` / `$web` | `en-us/1/menu/8244bf5cc5c74063ad55d2b9f992dda7.png` | `https://testcuassetsasa.blob.core.windows.net/$web/en-us/1/menu/8244bf5cc5c74063ad55d2b9f992dda7.png` | `2026-06-15T13:51:35Z` | `1043` | `application/octet-stream` |
| Online Web | Dev | `corp_menu_item_availability_image`, item `1000011`, `Web_ItemDetailImage` | `testcumobileimagesasa` / `new-menu` | `web/en-us/a3542c11efa342d9bc4942a116a58d3a.jpg` | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/web/en-us/a3542c11efa342d9bc4942a116a58d3a.jpg` | `2026-04-30T15:52:06Z` | `3204` | `application/octet-stream` |
| Online Web | Dev | `corp_menu_item_availability_image`, item `1000011`, `Web_MenuImage` | `testcumobileimagesasa` / `new-menu` | `web/en-us/9a63307456c747009240119804ab9f1d.jpg` | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/web/en-us/9a63307456c747009240119804ab9f1d.jpg` | `2026-04-30T15:52:08Z` | `6992` | `application/octet-stream` |
| Online Web | Test/QA | `corp_menu_item_availability_image`, item `1000057`, `Web_ItemDetailImage` | `testcumobileimagesasa` / `new-menu` | `web/en-us/e0d7ffe587c742448439e269bc433040.jpg` | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/web/en-us/e0d7ffe587c742448439e269bc433040.jpg` | `2026-06-15T13:51:37Z` | `6100` | `application/octet-stream` |
| Online Mobile | Test/QA | `corp_menu_item_availability_image`, item `1000057`, `Mobile_MenuImage` | `testcumobileimagesasa` / `new-menu` | `mobile/en-us/40898131355a44378ad1cf2009395e1e.jpg` | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/mobile/en-us/40898131355a44378ad1cf2009395e1e.jpg` | `2026-06-15T13:51:37Z` | `3204` | `application/octet-stream` |
| Online Mobile | Test/QA | Azure inventory sample not found in sampled DB rows | `testcumobileimagesasa` / `new-menu` | `mobile/en-ca/0021647c-98a4-41cc-8519-13f6b28c2fde.jpg` | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/mobile/en-ca/0021647c-98a4-41cc-8519-13f6b28c2fde.jpg` | `2026-04-08T14:04:09Z` | `84959` | `application/octet-stream` |
| Online Mobile | Prod | Azure inventory sample not found in sampled DB rows | `prodcumobileimagesasa` / `new-menu` | `mobile/en-ca/066df093-6e5b-4af5-a123-0daabd718249.jpg` | `https://prodcumobileimagesasa.blob.core.windows.net/new-menu/mobile/en-ca/066df093-6e5b-4af5-a123-0daabd718249.jpg` | `2026-04-20T17:45:47Z` | `37144` | `application/octet-stream` |
| Online Mobile | Prod | Azure inventory sample not found in sampled DB rows | `prodcumobileimagesasa` / `new-menu` | `mobile/en-ca/071a4b62-56a1-4153-b5d4-7c5946cacd43.jpg` | `https://prodcumobileimagesasa.blob.core.windows.net/new-menu/mobile/en-ca/071a4b62-56a1-4153-b5d4-7c5946cacd43.jpg` | `2026-04-20T17:44:00Z` | `459263` | `application/octet-stream` |
| Store/supporting asset | Prod | Azure inventory sample not found in sampled DB rows | `prodcuassetsasa` / `$web` | `en-ca/1/jobAid/CAN-12in_BBQ_Sauce_Extra.png` | `https://prodcuassetsasa.blob.core.windows.net/$web/en-ca/1/jobAid/CAN-12in_BBQ_Sauce_Extra.png` | `2024-02-26T11:52:52Z` | `37965` | `image/png` |
| Store/supporting asset | Prod | Azure inventory sample not found in sampled DB rows | `prodcuassetsasa` / `$web` | `en-ca/1/jobAid/CAN-12in_BBQ_Sauce_Light.png` | `https://prodcuassetsasa.blob.core.windows.net/$web/en-ca/1/jobAid/CAN-12in_BBQ_Sauce_Light.png` | `2024-02-26T11:52:53Z` | `32025` | `image/png` |

## Database-Derived Examples Not Verified In Azure

These examples come from current DB values and sampled filenames, but direct Azure verification either failed with RBAC, returned `BlobNotFound`, or was not attempted after a related access failure. They should not be treated as confirmed blob inventory until verified.

| Pattern | Environment | Full database-derived URL | Azure result |
| --- | --- | --- | --- |
| Store | Dev | `https://devcuassetsasa.blob.core.windows.net/$web/en-us/1/menu/7196e6f9abfe443ebdaf1159772d9c08.png` | Not verified in this pass. |
| Store | Dev | `https://devcuassetsasa.blob.core.windows.net/$web/en-us/1/menu/663155c84bfb4aa2a85858cc76ecf7c1.png` | Not verified in this pass. |
| Store | Prod | `https://prodcuassetsasa.blob.core.windows.net/$web/en-us/1/menu/0aca6e134b1343b69fe9170de5bc960c.png` | `BlobNotFound` at assumed `en-us` path. |
| Store | Prod | `https://prodcuassetsasa.blob.core.windows.net/$web/en-us/1/menu/f4c16d10ce0c43d596ef98c98301313b.png` | Not verified after related `BlobNotFound`; verify culture/path before automation. |
| Online Web | Test/QA | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/web/en-us/38d213a9badb4e4482b95816cdbefc92.jpg` | `BlobNotFound` at assumed `en-us` path. |
| Online Mobile | Test/QA | `https://testcumobileimagesasa.blob.core.windows.net/new-menu/mobile/en-us/101378_Orange.jpg` | `BlobNotFound` at assumed `en-us` path. |
| Online Web | Prod | `https://prodcumobileimagesasa.blob.core.windows.net/new-menu/web/en-us/77a727deea6e4d7083fa31af9ff9e62c.jpg` | `BlobNotFound` at assumed `en-us` path. |
| Online Mobile | Prod | `https://prodcumobileimagesasa.blob.core.windows.net/new-menu/mobile/en-us/60388e5d5da34655bbbd799f15ea6381.jpg` | `BlobNotFound` at assumed `en-us` path. |
| Deliverect | Dev | `https://devcudeliverectimages.blob.core.windows.net/read-bucket/en-us/c990aff653f94ea2b21bff3b71e8def4.jpg` | Blob read denied by RBAC on Deliverect storage. |
| Deliverect | Test/QA | `https://testcudeliverectimages.blob.core.windows.net/read-bucket/en-us/031dc39315114dc6a118bcc4dac0bf72.jpg` | Blob read denied by RBAC on Deliverect storage. |
| Deliverect | Prod | `https://prodcudeliverectimages.blob.core.windows.net/read-bucket/en-us/5034075e12df48b987b4295c9401fac3.jpg` | Not attempted after Deliverect RBAC failures; verify with storage RBAC before automation. |
| DMB | Dev | `https://devcudmbimages.blob.core.windows.net/read-bucket/fe69470a556d45ccb6c6056027a1434b.png` | Blob read denied by RBAC on DMB storage. |
| DMB | Test/QA | `https://testcudmbimages.blob.core.windows.net/read-bucket/1b15299290eb4be78229efd8a7ff48c5.png` | Blob read denied by RBAC on DMB storage. |
| DMB | Prod | `https://prodcudmbimages.blob.core.windows.net/read-bucket/2dec79b5b3e84d39a293dd011dbf0d4f.png` | Not attempted after DMB RBAC failures; verify with storage RBAC before automation. |

## Azure Blob Patterns Not Found In Sampled DB Rows

Direct storage inspection found patterns beyond the sampled menu-image rows.

| Account/container | Pattern | Evidence |
| --- | --- | --- |
| `testcumobileimagesasa` / `new-menu` | Hyphenated UUID JPGs under `mobile/en-ca/`. | Examples include `mobile/en-ca/0021647c-98a4-41cc-8519-13f6b28c2fde.jpg` and `mobile/en-ca/029a79ea-e8e2-4bc9-9e0a-3766df457b81.jpg`. |
| `prodcumobileimagesasa` / `new-menu` | Hyphenated UUID JPGs under `mobile/en-ca/`. | Examples include `mobile/en-ca/066df093-6e5b-4af5-a123-0daabd718249.jpg` and `mobile/en-ca/071a4b62-56a1-4153-b5d4-7c5946cacd43.jpg`. |
| `prodcumobileimagesasa` / `new-menu` | Non-image/test blob at container root. | `TestDocument.txt` exists with zero content length. |
| `prodcuassetsasa` / `$web` | Job aid PNGs under `en-ca/1/jobAid/`. | Examples include `CAN-12in_BBQ_Sauce_Extra.png` and `CAN-12in_BBQ_Sauce_Light.png`. |
| `prodcuassetsasa` / `$web` | Windows metadata files in content paths. | `en-ca/1/Thumbs.db` exists. |
| `testcumobileimagesasa` and `prodcumobileimagesasa` | Legacy or adjacent online-image containers. | Containers include `blobfiles`, `canblobfiles`, `canweborderingimages`, `gtblobfiles`, `gtweborderingimages`, `menuredesign`, `menuwarningimages`, `mexblobfiles`, `mexweborderingimages`, `nophotoimages`, `videos`, and `weborderingimages`. |
| `testcumobileimagesasa` | `integratedmenu` is not a container. | Direct container read returned `ContainerNotFound`; DB published roots referencing `.../integratedmenu/web` may be custom URL/path data, stale, or not represented as a container in this account. |
| Work accounts | Extra containers outside the DB work URL. | Dev has `$web`, `menu-creation`, and `read-bucket`; Test has `menu-creation` and `read-bucket`; Prod has `menu-creation`. Blob listing inside `menu-creation` was denied by RBAC. |

## Published-Menu Root URL Bases

`organization_as_menu_root` stores consumer-facing root image URL bases used by published menu channels. These rows are separate from upload work paths and final deployment destination rows.

| Environment | Observed published-menu root values |
| --- | --- |
| Dev | Online web/mobile roots point to `https://testcumobileimagesasa.blob.core.windows.net/integratedmenu/web` and `https://testcumobileimagesasa.blob.core.windows.net/integratedmenu/mobile`. Deliverect mostly points to `https://devcudeliverectimages.blob.core.windows.net/read-bucket`, with org 10 using an `es-gt` suffix. Azure verification found no `integratedmenu` container in `testcumobileimagesasa`. |
| Test/QA | Orgs 2, 6, and 11 online web/mobile roots point to `https://testcumobileimagesasa.blob.core.windows.net/new-menu/web/` and `https://testcumobileimagesasa.blob.core.windows.net/new-menu/mobile/`. Deliverect uses `https://testdeliverectimages.littlecaesars.com/read-bucket/...` culture paths for orgs 2, 6, 7, 10, and 11, and a bare read-bucket path for org 21. |
| Prod | Online, instore, and DMB published root columns are currently `NA`. Deliverect uses `https://prodcudeliverectimages.blob.core.windows.net/read-bucket/...` culture paths for orgs 2, 5, 6, 7, 10, and 11, a bare read-bucket path for org 21, and org 8 currently points to `https://testdeliverectimages.littlecaesars.com/read-bucket//`. |

## Related SQL Objects

| Object | Purpose |
| --- | --- |
| `system_attribute` | Stores `MenuCreationImageWorkeUrl`, the work-upload base URL. |
| `menu_create_deployment_root_organization_image` | Stores environment-specific final destination bases, path extras, language placement flags, image types, extensions, and sizes. |
| `organization_as_menu_root` | Stores published-menu channel/root image URL bases. |
| `api_menu_create_get_view_image_raw` | Joins applicable languages to deployment image rows, maps service-location/channel rules, normalizes URLs, computes work/final extras, and computes filename defaults. |
| `api_menu_create_get_view_image_upload` | Wraps image view data for upload consumers. |
| `api_menu_create_update_view_image` | Updates image metadata, audit, and validation after upload. |

Important routine fragments from `api_menu_create_get_view_image_raw`:

- URL IDs: `1 = Store`, `2 = Online Web`, `3 = Online Mobile`, `4 = Deliverect`, `5 = DMB`.
- Service-location routing: instore -> Store; instore DMB language -> DMB; online -> Online Web and Online Mobile; other service locations -> Deliverect.
- Final URL normalization: ensure `url_base` trailing slash; optionally add language before extra; add `url_add_extra`; optionally add language after extra.
- Work URL normalization: ensure `MenuCreationImageWorkeUrl` trailing slash and `menu-creation/` suffix; set `url_base_work_extra` from URL name; append language culture code.
- Filename logic: extract extension from existing save-as names when present; otherwise use `image_file_uuid` and set add-extension behavior.

## Related Code And Configuration

| File | Role |
| --- | --- |
| [MenuCreationDataController.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Api/Controllers/MenuCreationDataController.cs&_a=contents) | Defines `ImageUpdateView` and anonymous `SaveImage` endpoints. |
| [ImageUploadService.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Patch/ImageUploadService.cs&_a=contents) | Orchestrates upload flow, validates Images view id, enforces filename/extension rules, and updates DB metadata for UX-visible rows. |
| [BlobService.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Patch/BlobService.cs&_a=contents) | Uses `BlobContainerClient(new Uri(urlBase), new DefaultAzureCredential())`, creates the container if needed, and uploads/downloads blob name `/{path}{fileName}`. |
| [ImageDownloadService.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Patch/ImageDownloadService.cs&_a=contents) | Downloads images from blob storage for deployment/processing paths. |
| [MenuCreationDataService.Image.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/MenuCreationServices/MenuCreationDataService.Image.cs&_a=contents) | Calls image PostgreSQL routines with Dapper and deserializes `ImageUploadViewDTO` rows. |
| [ImageUploadViewDTO.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/DTO/MenuCreation/ImageUploadViewDTO.cs&_a=contents) | DB/API upload contract: image metadata, work URL, final URL, path extras, and visibility. |
| [UpdateImageViewDTO.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/DTO/MenuCreation/UpdateImageViewDTO.cs&_a=contents) | Frontend/API request contract for image updates. |
| [DeploymentSettings.cs](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/api/Lce.MenuManager.Core/Options/DeploymentSettings.cs&_a=contents) | Provides `EnvironmentId`; also contains legacy-looking `BlobImageStorageWork`/`BlobImageStorageFinal` properties not used by current `ImageUploadService`. |
| [values.yaml](https://lce-digital.visualstudio.com/eComm/_git/lce-api-menu?path=/charts/values.yaml&_a=contents) | Gives image/deploy endpoints 180 second Kong proxy timeouts. |

## Read-Only Evidence Queries And Commands

### PostgreSQL: Work Base

```sql
SELECT
    attribute_name,
    attribute_value
FROM public.system_attribute
WHERE attribute_name = 'MenuCreationImageWorkeUrl';
```

### PostgreSQL: Final Destination Rows

```sql
SELECT
    organization_id,
    environment_id,
    environment_name,
    url_id,
    url_name,
    url_base,
    url_add_language,
    url_add_language_after_extra,
    url_add_extra,
    array_agg(DISTINCT url_image_type ORDER BY url_image_type) AS image_types,
    array_agg(DISTINCT url_image_ext::text ORDER BY url_image_ext::text) AS image_extensions,
    array_agg(DISTINCT url_image_size::text ORDER BY url_image_size::text) FILTER (WHERE url_image_size IS NOT NULL) AS image_sizes
FROM public.menu_create_deployment_root_organization_image
WHERE environment_id = 2
GROUP BY
    organization_id,
    environment_id,
    environment_name,
    url_id,
    url_name,
    url_base,
    url_add_language,
    url_add_language_after_extra,
    url_add_extra
ORDER BY
    organization_id,
    url_id;
```

Change `environment_id` to `3` for Dev and `1` for Prod.

### PostgreSQL: Sample Filenames

```sql
WITH sampled AS (
    SELECT
        'corp_menu_item_availability_image' AS source_table,
        item_id::text AS source_id,
        service_location_id,
        image_id,
        image_type,
        image_file_name,
        updated_on
    FROM public.corp_menu_item_availability_image
    WHERE image_file_name IS NOT NULL
        AND btrim(image_file_name) <> ''
    UNION ALL
    SELECT
        'corp_menu_special_availability_image' AS source_table,
        special_id::text AS source_id,
        service_location_id,
        image_id,
        image_type,
        image_file_name,
        updated_on
    FROM public.corp_menu_special_availability_image
    WHERE image_file_name IS NOT NULL
        AND btrim(image_file_name) <> ''
    UNION ALL
    SELECT
        'dmb_product_image' AS source_table,
        xref_id::text AS source_id,
        1::smallint AS service_location_id,
        NULL::integer AS image_id,
        image_type,
        image_file_name,
        updated_on
    FROM public.dmb_product_image
    WHERE image_file_name IS NOT NULL
        AND btrim(image_file_name) <> ''
), ranked AS (
    SELECT
        *,
        row_number() OVER (
            PARTITION BY source_table, service_location_id, image_type
            ORDER BY updated_on DESC NULLS LAST, source_id
        ) AS rank_in_type
    FROM sampled
)
SELECT
    source_table,
    source_id,
    service_location_id,
    image_id,
    image_type,
    image_file_name,
    updated_on
FROM ranked
WHERE rank_in_type <= 2
ORDER BY
    source_table,
    service_location_id,
    image_type,
    rank_in_type;
```

### PostgreSQL: Published Root Bases

```sql
SELECT
    organization_id,
    country,
    instore_image_url_base,
    online_image_url_base_web,
    online_image_url_base_mobile,
    dmb_image_url_base,
    deliverect_image_url_base,
    updated_on
FROM public.organization_as_menu_root
ORDER BY
    organization_id;
```

### Azure: Account Discovery

Use Azure Resource Graph across all accessible subscriptions for expected storage account names. Record `name`, `subscriptionId`, `resourceGroup`, `location`, `kind`, `sku`, and `allowBlobPublicAccess`.

### Azure: Container Discovery

Use the storage MCP command `storage_blob_container_get` with `auth-method = Credential`, `subscription`, and `account`.

### Azure: Blob Verification

Use the storage MCP command `storage_blob_get` with `auth-method = Credential`, `subscription`, `account`, `container`, and exact `blob` name. Record `lastModified`, `contentLength`, `contentType`, `createdOn`, and status.

Do not use Azure Storage commands that upload, delete, create containers, regenerate keys, or change metadata while maintaining this document.