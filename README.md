# Rasa Train-Test Model GitHub Action

The GitHub action to run easily `rasa train` and `rasa test` in the CIs.

## Input Arguments

In order to pass the input parameters to the GH action, you have to use the [`with`](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepswith) argument in a step that uses the GH action, e.g.:

```yaml
jobs:
  my_first_job:
    name: My first job
    runs-on: ubuntu-latest
    steps:
      # Checks-out GitHub repository
      # more: https://github.com/actions/checkout
      - uses: actions/checkout@v2

      # Run rasa train and rasa test
      - name: My first step
        uses: RasaHQ/rasa-train-test-gha@main
        with:
          test_type: nlu
```

|        Input         |                                  Description                                  |          Default          |
| -------------------- | ----------------------------------------------------------------------------- | ------------------------- |
| `rasa_version`       | The Rasa Open Source version used to run test and train                       | `latest-full`             |
| `rasa_image`         | Custom Rasa Docker image. Useful if you use Rasa with custom Python modules.  | `none`                    |
| `rasa_train`         | Run `rasa train`                                                              | `true`                    |
| `rasa_test`          | Run `rasa test`                                                               | `true`                    |
| `data_validate`      | Validates domain and data files to check for possible mistakes                | `true`                    |
| `data_validate_args` | Additional arguments passed to the `rasa data validate` command               | `none`                    |
| `fine_tune`          | Fine-tune an existing model with new training dataset                         | `false`                   |
| `fixed_model_name`   | Name of the model file will be set to the given name                          | `none`                    |
| `workspace`          | The root directory containing your Rasa Open Source project                   | `${{ github.workspace }}` |
| `train_type`         | The types of training (available types: `core`/`nlu`/`all`)                   | `all`                     |
| `train_args`         | Additional arguments passed to the `rasa train` command                       | `none`                    |
| `test_type`          | The types of tests to run (available types: `core`/`nlu`/`all`)               | `all`                     |
| `test_nlu_args`      | Additional arguments passed to the `rasa test nlu` command                    | `none`                    |
| `test_core_args`     | Additional arguments passed to the `rasa test core` command                   | `none`                    |
| `publish_summary`    | Publish tests summary as a PR comment                                         | `true`                    |
| `github_token`       | GitHub Token - required to add a comment with summary                         | `none`                    |
| `configuration`      | Model configuration file                                                      | `config.yml`              |
| `model`              | Path to a file with a model. Use existing model instead of training a new one | `none`                    |
| `cross_validation`   | Switch on cross validation mode. Any provided model will be ignored           | `false`                   |
| `configuration_name` | Configuration name used in summary. If not provided a file name is used       | `none`                    |
| `data_name`          | Data name used in summary. If not provided a directory name is used           | `default`                 |
| `compare_report`     | Path to a report that will be used to compare results                         | `none`                    |
| `result_directory`   | Directory name where results are stored in                                    | `results`                 |
| `report_directory`   | Directory name where reports are stored in                                    | `reports`                 |
| `gomplate_image`     | Custom gomplate image. Useful if you use custom gomplate image                | `hairyhenderson/gomplate` |
| `tmpfs_directory`    | The directory location where tmpfs mounts                                     | `/.config`                |

## Outputs

The list of available output variables:

|  Output  |      Description      |
| -------- | --------------------- |
| `report` | Return report as JSON |

## Report

The GH action generates two reports, a report with a summary of run tests, the report is available as JSON and CSV file. The example of a report generated by the GH action can be found [here](examples/).

## Example Usage

In the example below, we are using [the Rasa Demo data](https://github.com/RasaHQ/rasa-demo).

```yaml
jobs:
    train_and_test:
        # ...
        steps:
            # ...
            - name: Train and Test Rasa Demo
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}
            # ...
```

The GitHub action by default adds a PR comment with summary (the summary comment can be disabled by setting the `publish_summary` input argument to `false`):

![Summary](_img/img_1.png)

### Use JSON report in GH workflow

The GH action returns a JSON report as an output. The following example shows how to use the output in a GH workflow.

```yaml
jobs:
    train_and_test:
        # ...
        steps:
            # ...
            - name: Train and Test Rasa Demo
              id: action
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}

            # We have to convert the output to JSON by using fromJSON built-in function
            # more: https://docs.github.com/en/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions#fromjson
            # syntax: fromJSON(steps.action.outputs.report).<data_name>[<configuration_name|configuration>]
            # example: fromJSON(steps.action.outputs.report).default['config.yml']
            - name: Check output
              if: fromJSON(steps.action.outputs.report).default['config.yml'].intent_classification.accuracy >= 0.8
              run: |
                echo "I'm doing extra work..."
                echo ${{ fromJSON(steps.action.outputs.report).default['config.yml'].intent_classification.accuracy }}
```

## Advanced examples

### Compare results with the base report

It possible to compare results to the other report, the feature is useful for example to see that a model is better than before changes. The difference against the report that we compare to is included in brackets.

```yaml
jobs:
    train_and_test:
        # ...
        steps:
            # ...
            - name: Train and Test Rasa Demo
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}
                # A path to the report that we want to compare to
                compare_report: 'report_to_compare.json'
                test_type: 'nlu'
            # ...
```

![Compare](_img/img_3.png)

### Use the existing model

It's possible to use the existing model instead of training a new one.

```yaml
jobs:
    test:
        # ...
        steps:
            # ...
            - name: Download Rasa X model
              run: |
                wget https://github.com/RasaHQ/rasa-x-demo/blob/0.33.0/models/model.tar.gz?raw=true \
                  -O test_model.tar.gz

            - name: Train and Test Rasa Demo
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}
                # If a file with the model is provided, training is disabled automatically
                model: test_model.tar.gz
            # ...
```

### Fine-tune the existing model

You can fine-tune an existing model with new training dataset. Please see [incremental training](https://rasa.com/docs/rasa/command-line-interface#incremental-training) for more details.

```yaml
jobs:
    train_and_test:
        # ...
        steps:
            # ...
            - name: Train and Test Rasa Demo
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}
                # By default, the number of epoches is defined in model configuration
                # You can shorten the epoches by using the --epoch-fraction flag
                fine_tune: 'true'
                train_args: >-
                  --epoch-fraction 0.5
                # ...
```

### Cross-validate NLU model

```yaml
jobs:
    train_and_test:
        # ...
        steps:
            # ...
            - name: Train and Test Rasa Demo
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}
                # Switch on cross validation mode. Any provided model will be ignored
                cross_validation: 'true'
                # Number of cross validation folds (cross validation only)
                # All available arguments: rasa test nlu --help
                test_nlu_args: '--folds 3'
                test_type: 'nlu'
            # ...
```

In a case where the cross-validation mode is enabled, a summary published as a PR comment includes `Intent Cross-Validation Results` and `Entity Cross-Validation Results`, e.g.

![Intent Cross-Validation Results](_img/img_4.png)

![Entity Cross-Validation Results](_img/img_5.png)

### Upload results and reports as artifacts

```yaml
jobs:
    train_and_test:
        # ...
        steps:
            # ...
            - name: Train and Test Rasa Demo
              id: action
              uses: RasaHQ/rasa-train-test-gha@main
              with:
                # List of available tags: https://hub.docker.com/r/rasa/rasa/tags
                rasa_version: '2.0.0-full'
                # In order to add a PR comment with summary
                # a GH Token has to be pass to the GH action
                github_token: ${{ secrets.GITHUB_TOKEN }}

            # The 'actions/upload-artifact' action to upload files
            # More: https://github.com/actions/upload-artifact
            - uses: actions/upload-artifact@v2
              with:
                name: rasa-demo-cfg
                path: |
                  results
                  reports
```
