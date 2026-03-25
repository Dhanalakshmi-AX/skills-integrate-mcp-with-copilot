This repository contains Microsoft Dynamics 365 Finance & Operations (D365 F&O) X++ extensions
and customizations. When reviewing code or responding to chat, apply the following D365 F&O–specific
best practices as a self-review checklist. Flag violations as inline review comments with a suggested fix.

1. Extension Model & Overlayering

NEVER use overlayering (direct modification of standard objects). All changes must use extensions:
_Extension classes, table extensions, form extensions, enum extensions, or CoC (Chain of Command).
Extension class names must follow the pattern: <StandardClass>_<ProjectPrefix>_Extension.
Do not call super() unless intentionally extending CoC behavior — always call super() in CoC
methods unless there is an explicit documented reason not to.
Avoid SysObsoleteAttribute methods in new code. Flag any usage.


2. X++ Query & SQL Performance

Never use nested while select loops when a join can replace them. Suggest a joined select statement.
Always specify only the fields actually needed in select statements — avoid select * from.
All select statements on large tables (LedgerJournalTrans, InventTrans, SalesLine, etc.) must
have a where clause that aligns with an existing or composite index. Flag any full-table scan risk.
Use firstOnly / firstOnly10 wherever a single or limited record is expected.
Prefer RecordInsertList or RecordSortedList over record-by-record insert in loops.
Flag any select inside a loop body as a performance risk — suggest moving it outside or using a map.
forupdate must only be used when the record is actually being modified. Flag unnecessary forupdate.


3. Transaction Integrity (ttsbegin / ttscommit)

Every ttsbegin must have a matching ttscommit or ttsabort. Flag unbalanced transaction blocks.
Do not nest ttsbegin / ttscommit unless the pattern is intentional and documented.
Avoid long-running transactions that span user interactions or batch waits.
Do not call external services (HTTP, Azure Service Bus, etc.) inside a ttsbegin block.


4. Exception Handling

All catch blocks must either re-throw, log, or handle the exception explicitly. Empty catch {} is not allowed.
Use error(), warning(), info() infolog methods appropriately — do not use throw for user-facing messages.
For batch jobs, always wrap the main logic in try/catch and log errors to the infolog.
Do not swallow Exception::Deadlock — always retry or re-throw.


5. Batch Framework Standards

All batch classes must extend RunBaseBatch or implement SysOperationServiceController.
canGoBatch() must return true for all classes intended to run in batch.
description() must return a meaningful, user-facing description (not empty or a class name).
pack() / unpack() must be implemented consistently — check that all #LOCALMACRO or container
members are included.
Do not call BatchHeader::addRuntimeTask() inside a transaction block.


6. Security & Data Access

Never bypass the SysQueryRangeUtil or skip security filters on queries.
Do not hardcode DataAreaId — always use curExt() or CompanyInfo context.
Flag any use of changeCompany — it must be wrapped in using and must not be used inside active transactions.
SysEntryPointAttribute must be present on all service/controller methods exposed to the UI or integration layer.


7. Naming Conventions

Classes: PascalCase with project prefix (e.g., PROJ_ClassName).
Methods: camelCase (e.g., processRecord, calculateTotal).
Variables: camelCase; avoid abbreviations except well-known D365 conventions (custTable, salesLine).
EDT fields on extensions: must carry the project prefix to avoid conflicts.
Labels: all user-visible strings must use labels (@LabelFile:LabelId) — no hardcoded strings.


8. Chain of Command (CoC) Best Practices

CoC methods must be protected unless there is a strong reason for public.
Always call next <methodName>(...) (i.e., super() equivalent) unless intentionally suppressing it — document why if suppressed.
Do not duplicate logic that already exists in the standard method being extended.
Flag CoC classes that do not have [ExtensionOf(classStr(...))] attribute.


9. Integration & External Services

HTTP calls and Azure Service Bus interactions must be async or run in batch — never inline during a form save or posting routine.
Always handle HTTP timeouts and retry logic. Flag any System.Net.Http.HttpClient usage without a timeout setting.
Sensitive values (connection strings, SAS tokens, client secrets) must be stored in Key Vault references via SysKeyVaultParameters — never hardcoded.
Log all integration failures to a custom error log table or Azure Application Insights.


10. Code Review Checklist (Apply to Every PR)
When reviewing a pull request, check each of the following and post an inline comment if violated:

 No overlayering — extensions only
 No select inside a loop
 All ttsbegin blocks are balanced
 No hardcoded strings — labels used
 No hardcoded DataAreaId
 Exception handling is not empty
 Batch classes implement pack/unpack
 CoC methods call next / document if suppressed
 No secrets or connection strings in code
 External service calls are not inside transaction blocks


Notes for Copilot

This is a D365 F&O X++ repository. Treat all .xpp files as X++ source code.
Apply all checks above as a self-review tool — flag issues as specific inline comments with a suggested correction.
Prioritize performance and transaction integrity issues as HIGH severity.
Flag naming violations and missing labels as MEDIUM severity.
Provide fixes in X++ syntax, not C# or any other language.
Limit review feedback to the changed lines in the PR diff unless a context issue requires broader reference.
