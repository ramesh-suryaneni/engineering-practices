**RBAC - authorization table design**

| Effect | Role | User ID | Resource | Action |
|--------|------|---------|----------|--------|
| allow/deny | admin, manager, executive | 123 | http://api.example.com/user | read_user_profile, update_user_profile, create_user_profile, delete_user_profile, * |
