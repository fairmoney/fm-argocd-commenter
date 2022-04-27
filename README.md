### ArgoCD-Commenter helm chart

`ArgoCD commenter` is a utility which syncs argocd apps with corresponding github repo
We use `ArgoCD commenter` to get notifications on `kube-state repo` from argocd,  it basically notifies us about the health of the applications running on  fairmoney `argocd`

# SetUP
  We need a organisation wide Github APP, which will work as a connection between `argocd` and `kube-state` (all apps are on `kube-state`).
  This app will give us three parameters which we use in `argocd-commenter` secret to estalish connection

  - appID: when we create this app we get an app id
  - InstallationID: we get this installation id by calling this API 
    `curl -s -H "Authorization: Bearer $JWT_TOKEN" -H "Accept: application/vnd.github.v3+json" https://api.github.com/app/installations`
  - PrivateKey: When we create this github APP we get an option to create private key for this app.
  
  The ruby script which creates this JWT token

  ```console
  require 'openssl'
  require 'jwt'  # https://rubygems.org/gems/jwt

  # Private key contents
  private_pem = File.read("/Users/apple/priv.pem") #Private key which was created at the time of creating this JWT token
  private_key = OpenSSL::PKey::RSA.new(private_pem)
  
  # Generate the JWT
  payload = {
    # issued at time, 60 seconds in the past to allow for clock drift
    iat: Time.now.to_i - 60,
    # JWT expiration time (10 minute maximum)
    exp: Time.now.to_i + (10 * 60),
    # GitHub App's identifier
    iss: "185678"
  }
  
  jwt = JWT.encode(payload, private_key, "RS256")
  puts jwt
  ```