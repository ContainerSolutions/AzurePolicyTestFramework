# Azure Policy Test Framework

This repository is a command line tool for testing Azure Policies.

## Usage

To use this framework, simply run a normal `go test call` passing the following variables:
```
export TEST_PATTERN="run-this-test|!not-this-one"
export TEST_CONFIG_PATH="test"
go test -v -timeout=30m ./policy_test.go
```

Any valid YAML within `TEST_CONFIG_PATH` folder will recursively be added as part of the test suite. Later on, the tests that are ran are filtered according to the chosen `TEST_PATTERN`. By default, `TEST_CONFIG_PATH="./"` and `TEST_PATTERN=".*"`

Here is an example YAML to configure the suite

```yaml
cases:
- name: test-deploy-if-not-exists
  setup:
    path: example/defaultSetup
    variables:
      name: "mypolicy"
      location: "westeurope"
      policy_rule_definition: "../../policy_definitions/vnet-deploy-subnet.rules.json"
      policy_params_definition: "../../policy_definitions/vnet-deploy-subnet.params.json"
      policy_params_values: |
        {
        "effect": {"value": "DeployIfNotExists"},
        "subnetAddress": {"value": "10.0.1.0/24"}
        }
  test:
    path: example/test
    variables:
        address_space: ["10.0.0.0/16"]
  after:
    waitBeforeRunning: "1m"
    path: example/after
- name: test-deny
  setup:
    path: example/defaultSetup
    variables:
      name: "mypolicy"
      location: "westeurope"
      policy_rule_definition: "../../policy_definitions/vnet-deploy-subnet.rules.json"
      policy_params_definition: "../../policy_definitions/vnet-deploy-subnet.params.json"
      policy_params_values: |
        {
        "effect": {"value": "Deny"},
        "subnetAddress": {"value": "10.0.1.0/24"}
        }
  test:
    path: example/test
    variables:
        address_space: ["10.0.0.0/16"]
    errorMessage: "mypolicy"
```

Each case is composed of a `setup`, a `test` and an `after` block. Only the `setup` block is mandatory. Each block accepts the same configruation:
```yaml
  path: string # path to terraform folder
  variables: {} #Optional - variables for the terraform apply
  varFiles: [] #Optional - list of var files for terraform apply
  errorMessage: string #Optional - If an error is expected, the error message to validate the correct behavior
  waitBeforeRunning: string #Optional - wait that many duration before running terraform Apply (for DeployIfNotExists tests)
```

## About Azure Policies

Azure Policy helps to enforce organizational standards and to assess compliance at-scale. Through its compliance dashboard, it provides an aggregated view to evaluate the overall state of the environment, with the ability to drill down to the per-resource, per-policy granularity. It also helps to bring your resources to compliance through bulk remediation for existing resources and automatic remediation for new resources.

Here are some resources about Azure Policies:

- [Policy Structure](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure)
- [Policy Effects](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/effects)
- [Policy Aliases](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure#aliases)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [Microsoft Contributor License Agreement](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow [Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
