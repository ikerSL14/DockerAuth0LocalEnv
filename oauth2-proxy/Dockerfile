FROM alpine:3.20 AS builder

ARG OAUTH2_PROXY_VERSION=v7.6.0
RUN apk add --no-cache curl tar ca-certificates \
    && curl -fsSL -o /tmp/oauth2-proxy.tar.gz \
       "https://github.com/oauth2-proxy/oauth2-proxy/releases/download/${OAUTH2_PROXY_VERSION}/oauth2-proxy-${OAUTH2_PROXY_VERSION}.linux-amd64.tar.gz" \
    && mkdir -p /tmp/oauth2-proxy \
    && tar -xzf /tmp/oauth2-proxy.tar.gz -C /tmp/oauth2-proxy --strip-components=1 \
    && install -m 0755 /tmp/oauth2-proxy/oauth2-proxy /usr/local/bin/oauth2-proxy

FROM alpine:3.20

RUN apk add --no-cache ca-certificates openssl apache2-utils && update-ca-certificates

COPY --from=builder /usr/local/bin/oauth2-proxy /usr/local/bin/oauth2-proxy

# Runtime entrypoint to render config with env vars and ensure valid cookie secret
RUN mkdir -p /etc/oauth2-proxy /entrypoint.d

RUN cat > /entrypoint.d/entrypoint.sh <<'EOF'
#!/bin/sh
set -eu

# Generate a valid 16-byte cookie secret if not provided
# oauth2-proxy v7.x expects base64-encoded string that decodes to 16, 24, or 32 bytes
# Using 16 bytes (24 base64 chars) to avoid issues with longer strings
# IMPORTANT: Must be exactly 16 bytes when decoded, which produces 24 base64 characters
MAX_ATTEMPTS=10
ATTEMPT=0
while [ $ATTEMPT -lt $MAX_ATTEMPTS ]; do
  if [ -z "${OAUTH2_PROXY_COOKIE_SECRET:-}" ] || [ $ATTEMPT -gt 0 ]; then
    # Generate exactly 16 random bytes and base64 encode them
    OAUTH2_PROXY_COOKIE_SECRET=$(openssl rand 16 | base64 | tr -d '\n')
  fi
  
  # Verify the secret decodes to exactly 16 bytes
  DECODED_LENGTH=$(echo -n "${OAUTH2_PROXY_COOKIE_SECRET}" | base64 -d 2>/dev/null | wc -c || echo "0")
  STRING_LENGTH=${#OAUTH2_PROXY_COOKIE_SECRET}
  
  if [ "$DECODED_LENGTH" = "16" ] && [ "$STRING_LENGTH" -le 24 ]; then
    # Valid secret found
    break
  fi
  
  # Invalid secret, regenerate
  OAUTH2_PROXY_COOKIE_SECRET=""
  ATTEMPT=$((ATTEMPT + 1))
done

# Final verification
DECODED_LENGTH=$(echo -n "${OAUTH2_PROXY_COOKIE_SECRET}" | base64 -d 2>/dev/null | wc -c || echo "0")
if [ "$DECODED_LENGTH" != "16" ]; then
  echo "ERROR: Failed to generate valid 16-byte cookie secret after $MAX_ATTEMPTS attempts" >&2
  exit 1
fi

echo "Generated cookie_secret: ${#OAUTH2_PROXY_COOKIE_SECRET} chars (decodes to ${DECODED_LENGTH} bytes)"

# Export as environment variable (oauth2-proxy can read from env vars)
export OAUTH2_PROXY_COOKIE_SECRET="${OAUTH2_PROXY_COOKIE_SECRET}"

# Write config for forward auth with Traefik using OIDC (Auth0)
# OIDC provider allows authentication with Auth0 users
{
  echo 'http_address = "0.0.0.0:4180"'
  echo "provider = \"${OAUTH2_PROXY_PROVIDER:-oidc}\""
  echo "client_id = \"${OAUTH2_PROXY_CLIENT_ID}\""
  echo "client_secret = \"${OAUTH2_PROXY_CLIENT_SECRET}\""
  echo "oidc_issuer_url = \"${OAUTH2_PROXY_OIDC_ISSUER_URL}\""
  echo "redirect_url = \"${OAUTH2_PROXY_REDIRECT_URL:-http://localhost/oauth2/callback}\""
  echo "cookie_secret = \"${OAUTH2_PROXY_COOKIE_SECRET}\""
  echo 'cookie_name = "_oauth2_proxy"'
  echo 'cookie_path = "/"'
  echo 'cookie_secure = false'
  echo 'cookie_httponly = true'
  echo 'cookie_samesite = "lax"'
  echo 'cookie_expire = "168h"'
  echo 'cookie_refresh = "1h"'
  echo 'skip_provider_button = true'
  echo 'reverse_proxy = true'
  echo 'email_domains = ["*"]'
  echo 'whitelist_domains = [".localhost", "localhost"]'
  echo 'pass_host_header = true'
  echo 'pass_basic_auth = true'
  echo 'insecure_oidc_allow_unverified_email = true'
  echo 'ssl_insecure_skip_verify = false'
  echo 'scope = "openid email profile"'
  echo 'skip_jwt_bearer_tokens = true'
  echo 'set_xauthrequest = true'
  echo 'pass_access_token = true'
  echo 'pass_user_headers = true'
} > /etc/oauth2-proxy/oauth2-proxy.cfg

# Verify cookie_secret in config is correct length (should be 24 chars for 16 bytes including padding)
echo "Verifying cookie_secret length..."
COOKIE_SECRET_FROM_CFG=$(grep "^cookie_secret" /etc/oauth2-proxy/oauth2-proxy.cfg | sed 's/.*= *"\(.*\)".*/\1/')
COOKIE_SECRET_LEN=${#COOKIE_SECRET_FROM_CFG}
DECODED_LEN=$(echo -n "${COOKIE_SECRET_FROM_CFG}" | base64 -d 2>/dev/null | wc -c || echo "0")
echo "Cookie secret in config: ${COOKIE_SECRET_LEN} chars, decodes to ${DECODED_LEN} bytes"

exec /usr/local/bin/oauth2-proxy --config=/etc/oauth2-proxy/oauth2-proxy.cfg "$@"
EOF

RUN sed -i 's/\r$//' /entrypoint.d/entrypoint.sh \
    && chmod +x /entrypoint.d/entrypoint.sh

EXPOSE 4180

ENTRYPOINT ["/entrypoint.d/entrypoint.sh"]

LABEL org.opencontainers.image.title="oauth2-proxy" \
      org.opencontainers.image.description="Reverse proxy that provides authentication with external providers" \
      org.opencontainers.image.source="https://github.com/oauth2-proxy/oauth2-proxy"

