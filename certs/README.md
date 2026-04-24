Place private or internal CA certificate material here only when `CERT_MODE=private`.

Expected files:

- `site.crt`: PEM-encoded server certificate, including intermediates when required
- `site.key`: PEM-encoded private key matching `site.crt`

The installer copies the selected files here and points Caddy at `/certs/site.crt` and `/certs/site.key`.

Do not commit real certificate material.
