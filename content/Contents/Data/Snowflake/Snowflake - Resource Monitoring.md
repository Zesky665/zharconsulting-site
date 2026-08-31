---
dg-publish: true
tags:
  - Snowflake
  - Monitoring
---

## Guidance Questions

1. What do resource monitors measure?
	1. Used credits.
	2. When resources like Virtual Warehouses run they consume credits, to help control this usage and prevent overconsumption, Snowflake has provided resource monitors. 
	3. [Source](https://docs.snowflake.com/en/user-guide/resource-monitors)
2. What actions can resource monitors trigger based on usage thresholds?
	1. Notify -> Just sends a notification.
	2. Notify and suspend -> Sends notification and suspends virtual warehouse as soon as it finishes all running processes.
	3. Notify and suspend immediately -> Sends notification and suspends virtual warehouse, drops all running processes. 
	- Multiple thresholds can be set with different actions on the same resource.
	- [Source](https://docs.snowflake.com/en/user-guide/resource-monitors#actions)
1. Who can create resource monitors?
	1. Users with ACCOUNTADMIN role.
	2. Other users if enabled by ACCOUNTADMIN.
	3. The SQL command used to create Monitors is:
		1. MONITOR
		2. MODIFY
	4. [Source](https://docs.snowflake.com/en/user-guide/resource-monitors#access-control-privileges-for-resource-monitors)
2. What are the scheduling options for resource monitor tracking and actions?
	1. Default: starts immediately upon monitor creation, resents at the beginning of each calendar month. 
	2. Frequency: The interval at which credits reset, relative to start date.
		1. Daily.
		2. Weekly.
		3. Monthly.
		4. Yearly.
		5. Never, reset, just keep accumulating until a certain threshold is reached. 
	3. Start: Time and Date when the interval starts:
		1. Immediately.
		2. Later, set the time and date yourself.
		- Resource monitors always reset at 12AM UTC.
		- If you put the start date on the last day of the month, snowflake automatically places it on the last day of all the subsequent months even if they have different numbers of days, also accounts for leap years.
	4. End: The time and date on which to suspend the virtual warehouse regardless of credit usage.
		1. Rarely used.
	5. [Source](https://docs.snowflake.com/en/user-guide/resource-monitors#schedule)
5. How are notifications enabled for resource monitors?
	1. In the Classic Console by an account admin level user.
	2. In Preferences >> Notifications.
	3. Before enabling email notifications, must verify email address.
	4. To enable notifications you must use the ACCOUNTADMIN role. 
	5. [Source](https://docs.snowflake.com/en/user-guide/resource-monitors#enabling-receipt-of-notifications-for-account-administrators)

## Quiz

1. Which of the following scenarios can Snowflake resource monitors be configured to support?
	1. Send a notification when a warehouse reaches a credit consumption threshold.
	2. Suspend a warehouse at a specific date and time, regardless of whether a credit quota threshold has been reached. 
	3. Monitor credit usage of warehouse with no interval end date, and do a different suspend action at 80% and 90% thresholds. 
2. Which statements are true regarding resource monitor creation and usage?
	1. Users with ACCOUNTADMIN role can create resource monitors. 
	2. Users can be granted privileges to view and modify resource monitors.
3. Which statements regarding Snowflake credit usage tracking are true?
	1. By default, credit usage tracking resets back to 0 at the beginning of each calendar month, matching the Snowflake billing cycle.
	2. By default, a resource monitor starts tracking assigned warehouse as soon as it is created, at the current timestamp.
	3. The schedule of a resource monitor can be customized to reset at an interval (such as daily, weekly, or annually) relative to a designated start date.
4. When a resource monitor action is set to "Suspend Immediately" and it's credit quota threshold is reached, additional credits may be consumed while it's assigned warehouses are being suspended. 
	1. True
5. Which must be done for resource monitor notifications to be received by account administrators?
	1. Account administrators must enable notifications in the web interface preferences.
	2. Each account administrator must provide and verify their email address. 