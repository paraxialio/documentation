# Code Scans

Paraxial.io uses RuboCop for static analysis. If a finding is a false positive, put a comment above the function to mark it as so:

```
# rubocop:disable Paraxial/SQL
def show
  User.find_by_sql("select * from users WHERE id = #{params[:id]} ")
end 
```

If the function has multiple findings, separate the cop name with a comma:

```
# rubocop:disable Paraxial/SQL, Paraxial/System
def show
  system(params[:id])
  User.find_by_sql("select * from users WHERE id = #{params[:id]} ")
end 
```

Some cops are built into RuboCop, and others and written by Paraxial.io. The full list: 

- Paraxial/Constantize
- Paraxial/CSRF
- Paraxial/HTMLSafe
- Paraxial/Raw
- Paraxial/Send
- Paraxial/SQL
- Paraxial/System
- Security/Eval
- Security/IoMethods
- Security/JSONLoad
- Security/MarshalLoad
- Security/Open
- Security/YAMLLoad
