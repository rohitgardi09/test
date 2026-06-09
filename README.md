Description:

Implemented a custom annotation-based API control mechanism to manage future deployments in the PreProd environment. This solution allows APIs to be independently enabled or disabled using annotations, ensuring that multiple future features can coexist in the same environment without affecting ongoing QA testing.

The annotation is applied at the API level and is intercepted through an Aspect. Before executing the API, the aspect validates the annotation configuration. If the API is enabled, the request proceeds normally; otherwise, the API execution is blocked and an appropriate response is returned.

Example:

@FeatureFlag(enabled = true)
@GetMapping("/existing-api")

→ API is accessible and available for testing.

@FeatureFlag(enabled = false)
@GetMapping("/future-api")

→ API remains disabled until it is ready for testing or release.

Scenario:

Future A is already deployed in PreProd and QA testing is in progress.

Future B is ready for deployment but is not yet ready for QA testing.

Both features can be deployed together in the same environment.

Future A APIs remain enabled for QA testing.

Future B APIs remain disabled using the annotation.

Once Future B is ready for testing, the annotation can be updated to enable the APIs.


Logic Flow:

API Request → Annotation Validation (Aspect) → Check enabled Flag →

true → Execute API

false → Block API Response


Benefits:

Supports parallel feature deployments in the same environment.

Prevents impact on ongoing QA testing.

Provides API-level control without modifying business logic.

Enables controlled exposure of future APIs.

Improves deployment flexibility and release management.
