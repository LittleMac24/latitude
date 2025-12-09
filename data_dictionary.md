### Data Dictionary Summary

Below is a summary of key fields used in the dataset:

- **created_at** — Timestamp indicating when the event itself occurred.  
- **event_type** — Categorical variable with two values: `system` and `user`, indicating how the event was triggered.  
- **event_name** — Contains 256 unique values. Seems to follow a `[Context]_[Object]_[Action]` structure. Next step: determine which correspond to user events vs. system events.  
- **event_variation** — Additional categorical descriptor that refines or specifies the meaning of `event_name`.  
- **user_id** — Identifier for the user who triggered or is associated with the event.  
- **metadata** — JSON-like structure containing device, location, scenario type, session ID, and other contextual attributes.  
- **year / month / day / hour** — Decomposition of `created_at` into separate time components.  
- **account_created_at** — Timestamp when a user account was first created.  
- **verified_at** — Timestamp when the user completed account verification or signup confirmation.


