# Intro / Basics
## Authentication with GCP
[Google Provider Configuration Reference | Guides | hashicorp/google | Terraform Registry](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/provider_reference)

1. Authenticate with GCP
	`gcloud auth login`
	Note: Above command opens a browser and login with your GCP credentials.
2. List the projects accessible by the current user
	`gcloud projects list`
3. Set the project
	`gcloud config set project PROJECT_ID`
4. List the current project
	`gcloud config get-value project`

# GCP Storage bucket
## Create n GCS Buckets
```sh
resource "google_storage_bucket" "buckets" {
  for_each                    = toset(var.bucket_name_list)
  name                        = each.key
  project                     = var.project_id
  location                    = var.location
  storage_class               = var.storage_class
  uniform_bucket_level_access = true
  labels = merge(
    { stage = var.stage },
    var.labels
  )

  force_destroy = lookup(
    var.force_destroy,
    lower(each.key),
    false,
  )

  versioning {
    enabled = lookup(
      var.versioning,
      lower(each.key),
      false,
    )
  }

  logging {
    log_bucket = google_storage_bucket.gcs_logging_bucket[0].name
  }

  dynamic "lifecycle_rule" {
    for_each = var.lifecycle_rules
    content {
      action {
        type          = lifecycle_rule.value.action.type
        storage_class = lookup(lifecycle_rule.value.action, "storage_class", null)
      }
      condition {
        age                   = lookup(lifecycle_rule.value.condition, "age", null)
        created_before        = lookup(lifecycle_rule.value.condition, "created_before", null)
        with_state            = lookup(lifecycle_rule.value.condition, "with_state", null)
        matches_storage_class = contains(keys(lifecycle_rule.value.condition), "matches_storage_class") ? split(",", lifecycle_rule.value.condition["matches_storage_class"]) : null
        num_newer_versions    = lookup(lifecycle_rule.value.condition, "num_newer_versions", null)
        matches_prefix        = lookup(lifecycle_rule.value.condition, "matches_prefix", null)
        matches_suffix        = lookup(lifecycle_rule.value.condition, "matches_suffix", null)
      }
    }
  }

}
```