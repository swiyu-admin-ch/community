# Expand-Migrate-Contract Pattern

In order to avoid breaking changes in the swiyu Trust Infrastructure we apply the Expand - Contract Pattern with an additional step. Expand-Migrate-Contract works by breaking down changes that are by themselves breaking, i.e. not backward compatible into three distinct, backward compatible phases.

![emc-pattern](https://github.com/swiyu-admin-ch/swiyu-admin-ch.github.io/blob/main/assets/images/EMC-Pattern.png)

## Chronological order

For the swiyu Public Beta Trust Infrastructure, the timeframe for the migrate step will be approximately one month. The expand step from our side will be announced as early as possible.

### Analysis and Preparation

- Identification of a breaking change and affected components
- Define if/when contract should take place
- Estimate impact and workload to complete the change and migration
- Consultations with community may also take place

### Start Expand Phase and Communication of EMC schema

- Announcements of breaking changes, affected components and indicative timeline
  - Online via [GitHub Announcements](https://github.com/orgs/swiyu-admin-ch/discussions/11) and [Release Notes](https://swiyu-admin-ch.github.io/release-announcements/)
  - If available, at Partizipationsmeeting 
- Announcements should take place 6 months prior to the breaking change (default - deviations may occur if needed)
  - Follow-up issue for contract step will be published on GitHub [StatusBoard](https://github.com/orgs/swiyu-admin-ch/projects/2/views/2)

### Migration Phase

- Default duration will be 3 months, deviations may occur if needed. 

### Contract

- Earliest after 3 months of migration (Note: extraordinary situations could demand shorter periods but this will be communicated accordingly)
- Before Contract is executed stakeholders may be consulted to ensure system stability
