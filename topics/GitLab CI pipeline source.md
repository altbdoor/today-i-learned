GitLab CI pipeline source
===

Nov 10, 2023

> **`CI_PIPELINE_SOURCE`**
>
> How the pipeline was triggered. Can be `push`, `web`, `schedule`, `api`, `external`, `chat`, `webide`, `merge_request_event`, `external_pull_request_event`, `parent_pipeline`, `trigger`, or `pipeline`.
>
> &mdash; https://docs.gitlab.com/ee/ci/variables/predefined_variables.html

This has been bugging me before, but it kinda resurfaced as one of the project I was working on required us to handle some specifics when running jobs against a merge request.
Some of the team members was super confused with this `CI_PIPELINE_SOURCE` variable, and it does not help that we have a general default of disabling `merge_request_event`.

```yml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE != 'merge_request_event'
```

To note, this rule above is redundant, as GitLab CI by default does not run pipelines for `merge_request_event`.

> Merge request pipelines:
> -  Do not run by default. The jobs in the CI/CD configuration file [must be configured](https://docs.gitlab.com/ee/ci/pipelines/merge_request_pipelines.html#prerequisites) to run in merge request pipelines.
> 
> &mdash; https://docs.gitlab.com/ee/ci/pipelines/merge_request_pipelines.html

I personally thought this default makes sense, since a `git push` on a merge request will trigger both `push` and `merge_request_event`, as [documented by GitLab too](https://docs.gitlab.com/ee/ci/jobs/job_control.html#avoid-duplicate-pipelines).

So let's assume a completely new GitLab repository, and a simple `.gitlab-ci.yml`:

```yml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE
    # (!) caution, this will cause duplicate pipelines!

stages:
  - step1

log-pipeline-source:
  stage: step1
  image: alpine:3
  script:
    - echo "The pipeline source is: $CI_PIPELINE_SOURCE"
```

| \# | Action | `$CI_PIPELINE_SOURCE` |
| --: | --- | --- |
| 1 | In `master`, add a new commit, and push `master` into origin | `push` |
| 2 | Create branch `test-mr` from `master`, and push `test-mr` into origin | `push` |
| 3 | In GitLab UI, create a merge request from `test-mr` into `master` | None |
| 4 | In `test-mr`, add a new commit, and push `test-mr` into origin | `push` and `merge_request_event` |
| 5 | Merge the merge request | `push` |

You might be asking, wouldn't (3) trigger a `merge_request_event`? Nope, GitLab CI is smart enough to check, that the **merge request pipeline** will only run if there are changes to the code.

Hence, with that in mind, we can conclude that:

1. `merge_request_event` is pretty rare, and
2. Only happens in a merge request that is updated

Depending on your project, this could raise some problems. Most notably, we could configure certain jobs to only run during a **merge request pipeline**,
but we'll have to make sure the depended jobs run as well!

```yml
# no workflow rules are set!

build-project:
  script:
    - echo "building..."

test-project:
  script:
    - echo "testing..."
  needs:
    - build-project
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - when: never
```

_P.S., the example above will run two pipelines!_

In the example above, we need to run `build-project`, before we can `test-project`. But adding the rule to run `test-project` for **merge request pipeline** alone,
would cause errors.

> **Unable to create pipeline**
>
> 'test-project' job needs 'build-project' job, but 'build-project' is not in any previous stage

If we have a complex network of `needs`... then it would be a nightmare to constantly check the depended job, to make sure that they have the same rule.

Sometimes I feel we should move away from configuring these YAML files manually, and instead [generate them on the fly...](https://engineering.grab.com/how-we-reduced-our-ci-yaml) Maybe a topic for some other day.

#### References

- https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- https://docs.gitlab.com/ee/ci/pipelines/merge_request_pipelines.html
- https://docs.gitlab.com/ee/ci/jobs/job_control.html#avoid-duplicate-pipelines
- https://engineering.grab.com/how-we-reduced-our-ci-yaml
