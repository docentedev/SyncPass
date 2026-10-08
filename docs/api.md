# login (vale username o email)
curl -X POST https://172-233-15-71.ip.linodeusercontent.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"juan_perez","password":"Password123"}'
# -> 200 {access_token, token_type: "Bearer", expires_in: 3600, user: {...}}