# Attest Flagged API

**Base URL:** `https://flagged-snapweb-attest.rusocial.io`

## `POST /sign`

Generates a base64-encoded Snapchat web attestation token. This token can be used to:

- Register Snapchat accounts via [accounts.snapchat.com/v2/signup](https://accounts.snapchat.com/v2/signup)
- Log in via [accounts.snapchat.com/v2/login](https://accounts.snapchat.com/v2/login)
- Access [snapchat.com/web](https://snapchat.com/web)

### Request

```json
{
  "username": "BvqMOcgHyY",
  "proxy": "http://user:pass@host:port"
}
```

| Field      | Type     | Required | Description                                                                                          |
|------------|----------|----------|------------------------------------------------------------------------------------------------------|
| `username` | `string` | Yes      | Snapchat username (1-15 chars). Must start with a letter. Only letters, numbers, `-`, `_`, or `.`.   |
| `proxy`    | `string` | Yes      | HTTP/HTTPS proxy URL for the bootstrap request.                                                      |

### Response `200`

```json
{
  "token_b64": "AQAB...",
  "username": "BvqMOcgHyY"
}
```

| Field       | Type     | Description                       |
|-------------|----------|-----------------------------------|
| `token_b64` | `string` | Base64-encoded attestation token. |
| `username`  | `string` | Username embedded in the token.   |

### Errors

| Status | Description                             |
|--------|-----------------------------------------|
| `413`  | Request body too large.                 |
| `422`  | Invalid or missing request body fields. |
| `500`  | Token construction failed.              |
| `502`  | Bootstrap request failed.               |

### Example Usage (C#)

**Fetch the attestation token:**

```csharp
async Task<string> GetAttestationTokenAsync(string username, string proxy)
{
    using var http = new HttpClient();
    var body = new StringContent(
        JsonSerializer.Serialize(new { username, proxy }),
        Encoding.UTF8,
        "application/json"
    );

    var resp = await http.PostAsync("https://flagged-snapweb-attest.rusocial.io/sign", body);
    resp.EnsureSuccessStatusCode();

    var json = JsonDocument.Parse(await resp.Content.ReadAsStringAsync());
    return json.RootElement.GetProperty("token_b64").GetString()!;
}
```

**gRPC-Web header handler:**

```csharp
internal class SnapWebHandler(
    string userAgent, string clientAgent, string chUa) : DelegatingHandler
{
    protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage req, CancellationToken ct)
    {
        req.Headers.Remove("User-Agent");

        var h = new Dictionary<string, string>
        {
            ["User-Agent"]                 = userAgent,
            ["x-snap-client-user-agent"]   = clientAgent,
            ["sec-ch-ua"]                  = chUa,
            ["sec-ch-ua-mobile"]           = "?0",
            ["sec-ch-ua-platform"]         = "Windows",
            ["sec-fetch-dest"]             = "empty",
            ["sec-fetch-mode"]             = "cors",
            ["sec-fetch-site"]             = "same-origin",
            ["x-grpc-web"]                 = "1",
            ["accept-language"]            = "en-US,en;q=0.9",
            ["referer"]                    = "https://accounts.snapchat.com/v2/signup"
        };

        foreach (var (k, v) in h)
            req.Headers.TryAddWithoutValidation(k, v);

        return base.SendAsync(req, ct);
    }
}
```

**Register an account:**

```csharp
var token = await GetAttestationTokenAsync("BvqMOcgHyY", "http://user:pass@host:port");

var grpcClient = new HttpClient(
    new GrpcWebHandler(GrpcWebMode.GrpcWeb,
        new SnapWebHandler(
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36",
            "SnapchatWeb/0.0.0.0 PROD (windows 10; chrome 146.0.0.0)",
            "\"Chromium\";v=\"146\", \"Not-A.Brand\";v=\"24\", \"Google Chrome\";v=\"146\""
        ) { InnerHandler = new HttpClientHandler() })
);

var reply = await client.WebRegisterAsync(new SCJanusWebRegisterRequest
{
    Email              = email,
    Username           = username,
    Password           = password,
    FirstName          = firstname,
    LastName           = "",
    TimeZone           = Config.TimeZone,
    IgnoreWelcomeEmail = false,
    RegistrationSource = SCJanusWebRegisterRequest.Types
        .SCJanusRegistrationSource.RegistrationSourceAccountsWeb,
    Birthdate = new SnapProto.Google.Type.GTPDate
    {
        Day   = rndDay,
        Month = rndMonth,
        Year  = rndYear,
    },
    WebRegistrationHeaderBrowser = new SCJanusWebRegistrationHeaderBrowser
    {
        Recapthcav3EntToken = recaptchatoken,
        SsoClientId         = "ads-api",
        ContinueParam       = "/accounts/sso?client_id=ads-api&referrer=https://ads.snapchat.com",
        MultiUser           = false,
    },
    AttestationPayload = ByteString.CopyFromUtf8(token)
});
```

---

To buy the **unflagged API**, join our Discord: [discord.gg/rusocial](https://discord.gg/rusocial)
