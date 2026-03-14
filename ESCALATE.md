# ESCALATE

> Human notification and approval protocol for AI agents operating in this repository.
> Spec version: 1.0 | Full specification: https://escalate.md

---

## TRIGGERS
# Conditions that REQUIRE human notification before the agent proceeds.
# Agent MUST pause and await approval unless marked auto_approve: true.

always_escalate:
  - deploy_to_production        # Any production deployment
  - send_external_communication # Emails, messages, social posts to real recipients
  - financial_transaction       # Any payment, transfer, or billing action
  - delete_data                 # Permanent deletion of user or business data
  - privilege_change            # Modifying access controls or permissions
  - new_third_party_integration # Connecting a new external service
  - cost_exceeds_usd: 100.00    # Single action estimated to cost over $100

auto_approve:
  - read_only_operations        # Reads, searches, lookups — no approval needed
  - internal_logging            # Writing to log files
  - draft_creation              # Creating drafts (not sending)

---

## CHANNELS
# Where and how to notify humans. Attempted in order until acknowledged.

channels:
  - type: email
    address: ops@example.com
    priority: 1
    timeout_minutes: 15

  - type: slack
    webhook: "${SLACK_ESCALATION_WEBHOOK}"
    channel: "#ai-alerts"
    priority: 2
    timeout_minutes: 10

  - type: sms
    number: "${OPS_PHONE_NUMBER}"
    priority: 3
    timeout_minutes: 5
    condition: no_response_from_above

---

## APPROVAL
# How the agent receives and validates human approval.

approval_methods:
  - reply_email                 # Reply to escalation email with "APPROVE" or "DENY"
  - slack_reaction              # React with ✅ (approve) or ❌ (deny)
  - api_endpoint: /approve      # POST to agent's approval endpoint with token

approval_timeout_minutes: 30    # If no response within this period, escalate further
on_timeout: escalate_to_killswitch  # Hand off to KILLSWITCH.md if unanswered
on_denial: halt_and_log         # Stop the proposed action, log reason if provided
on_approval: proceed_and_log    # Execute action, log approval with approver identity

require_reason_on_denial: false # Optionally require humans to explain denials

---

## CONTEXT
# What information the agent MUST include in every escalation notification.

include_in_notification:
  - action_requested            # Plain-English description of what the agent wants to do
  - reason                      # Why the agent believes this action is necessary
  - estimated_cost              # Cost estimate if applicable
  - reversibility               # Can this action be undone?
  - alternatives_considered     # What else the agent considered
  - session_id                  # For log correlation
  - timestamp                   # When the escalation was triggered
  - approval_deadline           # When the request expires

---

## AUDIT

log_file: .escalate.log
log_format: jsonl
log_fields:
  - timestamp
  - trigger_type
  - action_requested
  - notified_channels
  - response_received
  - response_time_minutes
  - approver_identity
  - outcome
  - session_id

---

## METADATA

owner: your-name-or-org
contact: ops@example.com
last_reviewed: 2026-03-10
review_frequency: quarterly
spec_version: "1.0"
spec_url: https://escalate.md
