## Non-Critical Issue

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Old OpenZeppelin version being used | 1 |

## [NC-1] Old OpenZeppelin version being used

The project is using a really old OpenZeppelin version (`4.9.0`) whereas the latest stable one the `4.9.3`
**Using an old version can lead to worse gas performance, known contract issues, and potential 0-day issues.**

Reference: https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts
