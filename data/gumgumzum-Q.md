### Updating `_SAFE_MODERATOR_HASH` or `_CONSOLE_FALLBACK_HANDLER_HASH` authorized addresses in `AddressProvider` will require every subAccount created under the previous ones to update their guard / fallback handler or their transactions will fail due to the checks in `TransactionValidator@_checkSubAccountSecurityConfig`

### subAccount owner check can be replaced in `ExecutorRegister@deRegisterExecutor` to be consistent with the one in `ExecutorRegister@registerExecutor`

### A subAccount executor can be the subAccount itself or the console account

### A console account can be deployed with one of his future subAccounts as one of its owners