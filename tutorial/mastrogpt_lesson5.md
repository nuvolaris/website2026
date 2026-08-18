# Lesson 5 — Computer Vision & Object Storage (MastroGPT • Apache Open Serverless)

In this lesson you’ll analyze images with an **open‑source VLM** (LLaMA Vision via Ollama) and manage files with **S3‑compatible storage**. You’ll build:
- A **vision action** that accepts an uploaded image (form) and describes it.
- A **storage action** that uploads, lists, deletes, and analyzes images stored in S3 using **signed public URLs** for display.

---

## 0) Start & fetch Lesson 5

1. Open your **starter** (fork) on GitHub. If prompted, **Sync fork → Update branch**.  
2. Launch your **Codespace** (or local env).  
3. In VS Code, log in via the 💬 extension and download **Lesson 5** (PDF + Markdown).

---

## 1) What you’ll learn

- How to call a **Vision LLM** (LLaMA Vision) with images encoded as **Base64**.  
- How to build an action that **shows a file-upload form**, reads the image, and **renders HTML** with the image preview + description.  
- How to use **S3‑compatible storage** (internal/test/external endpoints), and when to generate **signed URLs** for public display.  

---

## 2) Vision fundamentals (Ollama Chat API)

Vision models accept a **message** that includes an image payload in Base64. The flow:

1. **Create the base URL** to your Ollama endpoint (auth is usually embedded in the URL).
2. **Read an image file** and encode to Base64.
3. **POST** to the vision/chat endpoint with `model`, a `messages` array, and the **image** in the content.  
4. If streaming: **iterate** the response chunks; otherwise **collect** to a final string.

### Sketch (Python)
```python
import base64, json, requests

def b64(path):
    return base64.b64encode(open(path, "rb").read()).decode()

def vision_describe(base_url, model, path):
    img64 = b64(path)
    url = f"{base_url}/api/chat"       # example: Ollama chat API
    body = {
        "model": model,                # e.g., "llama3.2-vision"
        "messages": [{
            "role": "user",
            "content": [
                {"type":"text","text":"Describe this image."},
                {"type":"image_url","image_url":{"url": f"data:image/png;base64,{img64}"}},
            ],
        }],
        "stream": True
    }
    with requests.post(url, json=body, stream=True, timeout=300) as r:
        parts = []
        for line in r.iter_lines():
            if not line: 
                continue
            # each line is a JSON object; extract partial text
            obj = json.loads(line.decode("utf-8"))
            parts.append(obj.get("message",{}).get("content","") or obj.get("response",""))
        return "".join(parts)
```

---

## 3) A simple **Vision action** with a file‑upload form

### Return a file form (first request)
```python
def main(args):
    if not args.get("form"):
        form = [{
            "name": "pic",
            "label": "Upload image",
            "kind": "file",
            "accept": ["image/png","image/jpeg"]
        }]
        return {"output":"Please upload an image","form":form}
```

### Handle the submitted form
```python
import html

def main(args):
    data = args.get("form")
    if not data:
        # return form as above
        ...
    pic = data.get("pic")              # already Base64 (data URL or raw base64 per your setup)
    if not pic:
        return {"output":"No image provided"}

    # Call your vision helper (non‑streaming shown for brevity)
    desc = vision_describe_from_b64(args, pic)

    # Render an inline preview using data URL + a caption
    img_html = f'<img alt="uploaded" src="{html.escape(pic if pic.startswith("data:") else "data:image/png;base64,"+pic)}" style="max-width:480px;border-radius:12px" />'
    return {"output": desc, "html": img_html}
```

> Pinocchio’s display can render simple HTML fragments returned by your action.

---

## 4) S3‑compatible storage — internal, test, external

There are **three endpoints** you’ll encounter:

- **Internal (production runtime)**: used by deployed actions. (`S3_HOST` + `S3_PORT`)  
- **Test (dev/editor)**: used by tests/CLI inside Codespace. (`test.env`)  
- **External (public)**: used to **expose objects via the web** (signed URL). (`S3_API_URL`)

Keep them straight:
- Internal/Test → read/write via SDK (private).  
- External → generate a **signed URL** so the browser can display the file publicly for a short time.

### Minimal S3 client sketch
```python
import boto3, os

def mk_s3(args):
    # Choose host/creds depending on context (test vs runtime)
    endpoint = args.get("S3_ENDPOINT") or os.getenv("S3_ENDPOINT")
    key      = args.get("S3_ACCESS_KEY") or os.getenv("S3_ACCESS_KEY")
    secret   = args.get("S3_SECRET_KEY") or os.getenv("S3_SECRET_KEY")
    region   = os.getenv("AWS_REGION","us-east-1")
    return boto3.client("s3",
        endpoint_url=endpoint,
        aws_access_key_id=key,
        aws_secret_access_key=secret,
        region_name=region,
    )
```

### Common operations
```python
def s3_put(s3, bucket, key, bytes_):  s3.put_object(Bucket=bucket, Key=key, Body=bytes_)
def s3_get(s3, bucket, key):          return s3.get_object(Bucket=bucket, Key=key)["Body"].read()
def s3_list(s3, bucket, prefix=""):   return s3.list_objects_v2(Bucket=bucket, Prefix=prefix).get("Contents",[])
def s3_del(s3, bucket, key):          s3.delete_object(Bucket=bucket, Key=key)
```

### Signed (public) URL
```python
def s3_signed_url(s3, bucket, key, expires=3600, external_base=None):
    url = s3.generate_presigned_url(
        "get_object",
        Params={"Bucket": bucket, "Key": key},
        ExpiresIn=expires,
    )
    # If your presign uses an internal endpoint, rewrite host to the external base
    if external_base:
        # naive replace; adapt to your deployment DNS
        from urllib.parse import urlparse
        p = urlparse(url)
        url = url.replace(f"{p.scheme}://{p.netloc}", external_base.rstrip("/"))
    return url
```

---

## 5) Storage action — upload, list, delete, analyze

Support three command styles in `input`:
- `*<substr>` → **list** objects matching substring
- `!<prefix>` → **delete** objects with that prefix
- `@<key>`    → **analyze** an S3 object with Vision and show a **public preview**

```python
def main(args):
    s3   = mk_s3(args)
    bkt  = args.get("S3_BUCKET")
    ext  = args.get("S3_API_URL")      # external public base
    text = (args.get("input") or "").strip()

    if text.startswith("*"):
        substr = text[1:]
        objs = s3_list(s3, bkt)
        names = [o["Key"] for o in objs if substr in o["Key"]]
        return {"output": "\n".join(names) or "(no matches)"}

    if text.startswith("!"):
        prefix = text[1:]
        objs = s3_list(s3, bkt, prefix=prefix)
        for o in objs:
            s3_del(s3, bkt, o["Key"])
        return {"output": f"Deleted {len(objs)} object(s) with prefix '{prefix}'"}

    if text.startswith("@"):
        key = text[1:]
        data = s3_get(s3, bkt, key)
        import base64
        img64 = base64.b64encode(data).decode()
        desc  = vision_describe_from_b64(args, img64)
        pub   = s3_signed_url(s3, bkt, key, expires=3600, external_base=ext)
        html  = f'<img alt="{key}" src="{pub}" style="max-width:480px;border-radius:12px" />'
        return {"output": desc, "html": html}

    return {"output": "Usage: *substr to list, !prefix to delete, @key to analyze"}
```

---

## 6) End‑to‑end flow you should have

1. **Vision Form Action**  
   - First request → returns a **file form**.  
   - Submit → analyzes the image (Base64) and returns **description + HTML preview**.

2. **S3 Storage Action**  
   - Upload images directly to S3 (via UI upload or separate tool).  
   - `*substr` → list matches.  
   - `!prefix` → delete by prefix.  
   - `@key` → analyze from storage and display with a **signed public URL**.

---

## 7) Exercises

1. **Unify**: modify the **form action** so that it **uploads** the image to S3 first, then:  
   - Uses **Base64** only for the vision request (from the freshly uploaded object), and  
   - Displays the image via a **signed external URL** (not inline Base64).

2. **Harden**: validate MIME types and size before upload; handle missing keys gracefully.  

3. **Extend**: add a `#search cats` command that lists only images whose **vision description** matches a term (store short descriptors in S3 object metadata or Redis).

---

## 8) Gotchas & tips

- Keep straight the **internal/test/external** S3 endpoints.  
- **Signed URLs** expire → choose a sensible TTL (e.g., 1 hour).  
- For streaming, make your action return `{ "streaming": true }` on first response so the UI switches to stream mode.  
- Prefer **small preview sizes** (server‑side resize) when showing many images.  

---

You now have the building blocks for **vision‑enabled, storage‑backed** assistants. 🚀
