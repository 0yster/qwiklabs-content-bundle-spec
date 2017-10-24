# Qwiklabs Lab Bundle Specification

> This is a DRAFT document and refers to QLB schema v1-prerelease.
>
> We welcome feedback as this format evolves.

A Qwiklabs Bundle, or QLB, is any directory that contains a qwiklabs.yaml file which defines a learning entity in Qwiklabs. This specification focuses specifically on the Lab entity.

The QLB format aims to:

1. Make it easy to author developer learning materials in your native tools.

2. Provide a simple format into which existing content can be programmatically transformed, making it convenient to migrate existing learning materials from diverse sources into Qwiklabs modules.

## Design Goals

### Localization is a first class construct

We understand that not all instructional content is localized... but it should be! Furthermore, localization requires a lot of work, so we want to make it as easy as possible to add localized content to a lab bundle.

Therefore, QLB will prioritize explicit and clear localization semantics at the risk of being overly verbose for authors who only work with one locale.

For instance, `locales` is a required field for external resources. If you only have one locale you still need to specify it as the `default` locale:

```yml
- type: github
  ref: code_repo
  locales:
    default:
      title: Self-referential Github Repo
      uri: https://github.com/CloudVLab/qwiklabs-lab-bundle-spec/tree/v1-prerelease
```

> TODO: Explain `default` locale.

### Prefer explicit configuration over convention

The only requirement we put on your bundle's file structure is the `qwiklabs.yaml` MUST be in the root folder of the bundle. You can arrange your other files in whatever folder structure you choose.

The lab definition in `qwiklabs.yaml` explicitly references files when specifying instructions, resources, etc. This makes it much easier to do static validation of your package and ensure that a file you intended to be there isn't missing.

> TODO: Add Dave's example of forgetting to add a file to a git repo, thus loosing an entire locale.

### Schema Version will be trackedthat

> TODO:
> - v1 the spec will evolve in an "additive" manner (e.g. new resource types)
> - Breaking changes will result in a new version change.
> - Older schema versions will be supported for a "reasonable" deprecation periods.
> - The Qwiklabs team will try to create migration tools that make schema updates a mostly automated process

## `qwiklabs.yaml` Structure

Here's a sample `qwiklabs.yaml` file, with all nested details removed to make it easier to see the general file structure.

```yml
entity_type: Lab
schema_version: 1

# Lab Attributes
name: my-awesome-lab
duration: 60
level: intro
tags: [sample, life-changing, gcp]
# ...

# Explicit specification of instruction
instructions:
  # ...

resources:
  # ...

environment_resources:
  # ...

assessment:
  # ...
```

Two properties are critical for specifying your lab bundle:

- `entity_type`

  Currently this can only be "Lab". In the future we will expand the QLB specification to include "Courses", "Quests", and other content that Qwiklabs supports.

- `schema_version`

  TODO

### Lab attributes

attribute | required | type      | notes
--------- | -------- | --------- | -----------------------------------------------------------------------------
name      | ✓        | string    | Identifier for this lab, must be unique per "library" (think github org/repo)
duration  | ✓        | integer   | Amount of time the user is expected to have (minutes)
level     |          | string    | enum?
logo      |          | file path |
tags      |          | array     |

```ruby
# NOTE: Unreferenced properties on Lab model that we may also want to add here.
t.string   "creator_tagline",                           limit: 255
t.decimal  "price",                                                      precision: 8, scale: 2
t.boolean  "display_connection_fleetconsole"
t.boolean  "display_connection_ssh"
t.boolean  "display_connection_vnc"
t.boolean  "display_connection_rdp"
t.boolean  "display_connection_custom"
t.boolean  "display_connection_access_key_id"
t.boolean  "send_end_lab_email",                                                                 default: true
t.boolean  "concurrent_allowed",                                                                 default: true
```

### Instructions

attribute | required | type       | notes
--------- | -------- | ---------- | -------------------------------------------------------------------
type      | ✓        | enum       | [See list of valid types below]
locales   | ✓        | dictionary | Keys are locale codes, the values are paths to files in the bundle.

```yml
instruction:
  type: markdown
  locales:
    default: "./instructions/en.md"
    es: "./instructions/es.md"
```

> TODO: Document the meta data specified in the instruction file.

### Resources

Resources are additional materials that learners may refer to while taking this lab.

We encourage content authors to use as few external links as possible. Qwiklabs cannot guarantee that those links will be available when a learner takes your lab. For instance, if you have a PDF that you wish to include in the lab, you should add it as a file in this lab bundle instead of referencing it as a link to Cloud Storage or S3.

TODO: Recommendation for extremely large resources.

> **Note**: If you are linking to an external resource that has its own understanding of source control, please link to the specific revision of that resource. That way, if the external resource is updated, your learners will not be affected. For example, if you are referencing a Github repo, include the link to a specific tag, instead of the default branch.
>
> Brittle: <https://github.com/CloudVLab/qwiklabs-lab-bundle-spec/>
>
> Better: <https://github.com/CloudVLab/qwiklabs-lab-bundle-spec/tree/v1-prerelease>

attribute | required | type        | notes
--------- | -------- | ----------- | ---------------------------------------------------------------------------------
type      | ✓        | enum        | [See list of valid types below]
ref       |          | string      | Identifier that can be used throughout project bundle
locales   | ✓        | dictionary  | Keys are locale codes, the values are dictionaries with the following attributes:
-- title   | ✓        | string      | Display title for the resource
-- uri     | ✓        | URL or path | External URL or path to local file in bundle

```yml
resources:
  - type: github
    ref: code_repo
    locales:
      default:
        title: Self-referential Github Repo
        uri: https://github.com/CloudVLab/qwiklabs-lab-bundle-spec/tree/v1-prerelease
      es:
        title: Auto-referencial Github Repo
        uri: https://github.com/CloudVLab/qwiklabs-lab-bundle-spec/tree/v1-prerelease
  - type: file
    locales:
      default:
        title: Sample PDF
        uri: "./resources/sample-en.pdf"
      es:
        title: Ejemplo de PDF
        uri: "./resources/sample-es.pdf"
```

#### Valid types

- `file` - a relative path to a file in the lab bundle
- `link` - a url to an external resource
- `github` - Github link
- `youtube` - Youtube link

> TODO:
> - Should we auto-detect URLs we display in a special manner, or special external URL types or have the author specify their type explicitly?
> - Does the fact that we know it's a Github repo give us any additional functionality when referencing it elsewhere in the bundle?

### Environment Resources

attribute | required | type   | notes
--------- | -------- | ------ | -----------------------------------------------------
type      | ✓        | enum   | [See list of valid types]
ref       |          | string | Identifier that can be used throughout project bundle

> **Note:** Additional valid properties of the asset depend on the selected type.

```yml
environment_resources:
  - type: gcp_project
    ref: my_primary_project
    dm_script:
      locales:
        default: "./deployment_manager/instance_pool-en.py"
        es: "./deployment_manager/instance_pool-es.py"
  - type: gcp_user
    ref: primary_user
    permissions:
      - my_primary_project:
        - compute: editor
  - type: gcp_user
    ref: secondary_user
```

#### Valid types

##### AWS Account (aws-account)

attribute | required | type           | notes
--------- | -------- | -------------- | -----------------------------------------------------
cf_script |          | localized_path | relative path to file in bundle
variant   |          | enum           | TODO: This maps to our current understanding of fleet

```yml
- type: aws-account
  ref: primary_account
  variant: STS
  cf_script:
    locales:
      default: ./cloudformation/intro-dynamo-db.yaml
      es: ./cloudformation/intro-dynamo-db-es.yaml
```

```ruby
# NOTE: Unreferenced properties on Lab model that we may also want to add here.
t.decimal  "billing_limit",                                              precision: 8, scale: 2, default: 0.0
t.string   "aws_region",                                limit: 255
t.boolean  "global_set_vnc_password",                                                            default: true
t.boolean  "allow_aws_vpc_deletion",                                                             default: false
t.boolean  "allow_aws_subnet_deletion",                                                          default: false
t.boolean  "allow_aws_spot_instances",                                                           default: false
t.string   "allow_aws_ondemand_instances",              limit: 255
t.boolean  "allow_aws_dedicated_instances",                                                      default: false
t.string   "allow_aws_rds_instances",                   limit: 255
```

##### GCP Project (gcp-project)

attribute | required | type           | notes
--------- | -------- | -------------- | -----------------------------------------------------
dm_script |          | localized_path | relative path to file in bundle
variant   |          | enum           | TODO: This maps to our current understanding of fleet

```yml
- type: gcp-project
  ref: secondary_project
  variant: ASL
  dm_script:
    locales:
      default: ./deployment_manager/instance_pool.yaml
      es: ./deployment_manager/instance_poolb-es.yaml
```

```ruby
# NOTE: Unreferenced properties on Lab model that we may also want to add here.
t.string   "cloud_region",                              limit: 255
```

##### GCP User (gcp-user)

attribute   | required | type       | notes
----------- | -------- | ---------- | -----------------------------------------------------
credentials |          | boolean    | Should the learner be given credentials for this user
roles       |          | dictionary | Specificy IAM roles per project

```yml
- type: gcp-user
  ref: learner_user
  credentials: true
  roles:
    # Reference to a GCP project asset
    my_project:
      - project.editor
    other_project:
      - pubsub.editor
      - storage.admin
```

##### GSuite Domain (gsuite-domain)

> TODO: Ask @davetchen what defines a GSuite fleet

### Assessment

> TODO