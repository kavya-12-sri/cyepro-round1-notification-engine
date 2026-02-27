# Notification Prioritization Engine  
Round 1 – Cyepro Solutions  

---

## 1. Problem Understanding

Applications send notifications from multiple services:
- Messages
- Alerts
- Reminders
- Promotions
- System events

Users receive too many notifications.

The system must decide for each event:
- NOW → Send immediately
- LATER → Schedule
- NEVER → Suppress

The solution must:
- Prevent duplicates
- Reduce alert fatigue
- Handle conflicting priorities
- Support configurable rules
- Provide explainable decisions
- Fail safely

---

## 2. High-Level Architecture

Incoming Event  
↓  
API Gateway  
↓  
Validation Service  
↓  
Deduplication Engine  
↓  
Decision Engine  
↓  
Scheduler (if Later)  
↓  
Notification Dispatcher  
↓  
User  

Side Components:
- User History DB
- Rule Engine
- Audit Log DB

---

## 3. Decision Logic

1. Expiry Check  
If expired → NEVER  

2. Duplicate Check  
If duplicate → NEVER  

3. Urgency Check  
If high priority and time-sensitive → NOW  

4. Alert Fatigue Check  
If hourly/daily limit exceeded → LATER  

5. Promotional Filtering  
If promotional during noisy period → LATER or NEVER  

6. Default  
Send NOW  

---

## 4. Data Model

### NotificationEvent
- event_id
- user_id
- event_type
- message
- priority_hint
- timestamp
- expires_at
- channel
- dedupe_key

### UserNotificationHistory
- user_id
- last_notification_time
- hourly_count
- daily_count

### SuppressionRecord
- event_id
- suppression_reason
- timestamp

### AuditLog
- event_id
- decision
- explanation
- decision_time_ms
- fallback_used

---

## 5. API Interfaces

POST /events  
GET /audit/{event_id}  
GET /user/{user_id}/history  
POST /rules  
GET /health  

---

## 6. Duplicate Prevention Strategy

Exact Duplicate:
- Use dedupe_key
- Or generate hash(user_id + event_type + message)
- Store in Redis (10 min TTL)

Near Duplicate:
- Compare with last 5 notifications
- If similarity > 85% → Suppress

---

## 7. Alert Fatigue Strategy

- Max 5 notifications per hour
- Max 20 per day
- 5-minute cooldown
- Quiet hours (10PM–7AM)
- Batch low priority notifications into digest

---

## 8. Fallback Strategy

If AI is slow/unavailable:
- Timeout 200ms
- Switch to rule-based engine
- Retry queue
- Never drop high priority notifications
- Log fallback reason

---

## 9. Metrics & Monitoring

- Decision latency
- Suppression rate
- Duplicate rate
- Delivery success rate
- System uptime

Monitoring:
- Prometheus
- Grafana
- Central logging

---

## 10. Tradeoffs

- AI accuracy vs latency
- Strict suppression vs missing important alerts
- More rules increase complexity

---

## 11. Conclusion

This solution provides a scalable, explainable, and fault-tolerant notification prioritization system that balances urgency and user experience.
