# How to Run Rust on Lambda

* [Function Code](#function-code)
* [Getting Started](#getting-started)
  * [Start Development Server with Watch Command](#start-development-server-with-watch-command)
  * [Bundle Artifacts with Build Command](#bundle-artifacts-with-build-command)
* [Deploy to AWS with Deploy Command](#deploy-to-aws-with-deploy-command)
  * [Send a Request with Invoke Command](#send-a-request-with-invoke-command)

## Function Code

```rs
// src/main.rs

use lambda_http::{run, service_fn, Body, Error, Request, RequestExt, Response};

async fn function_handler(event: Request) -> Result<Response<Body>, Error> {
    let resp = Response::builder()
        .status(200)
        .header("content-type", "text/html")
        .body("<h2>Hello from LogRocket</h2>".into())
        .map_err(Box::new)?;
    Ok(resp)
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::INFO)
        .with_target(false)
        .without_time()
        .init();
    run(service_fn(function_handler)).await
}
```

## Getting Started

Install [`cargo`](https://doc.rust-lang.org/cargo/) with [`rustup`](https://doc.rust-lang.org/cargo/getting-started/installation.html) and [`cargo-lambda`](https://www.cargo-lambda.info/guide/installation.html).

```bash
curl https://sh.rustup.rs -sSf | sh
brew tap cargo-lambda/cargo-lambda
brew install cargo-lambda
```

### Start Development Server with Watch Command

Boot a development server with the [`watch` subcommand](https://www.cargo-lambda.info/commands/watch.html).

```bash
cargo lambda watch
```

Open [localhost:9000/lambda-url/rustrocket](http://localhost:9000/lambda-url/rustrocket) to invoke and view your function in the browser.

![hello from logrocket](https://user-images.githubusercontent.com/12433465/190524707-2ae17b2b-a5c2-4cf8-ae75-a1b8a52c7b15.png)

### Bundle Artifacts with Build Command

Compile your function natively with the [`build` subcommand](https://www.cargo-lambda.info/commands/build.html).

```bash
cargo lambda build
```

## Deploy to AWS with Deploy Command

Upload your function to AWS Lambda with the [`deploy` subcommand](https://www.cargo-lambda.info/commands/deploy.html).

```bash
cargo lambda deploy --enable-function-url
```

If this command fails then you will need to create an AWS IAM role. Install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and run the following command:

```bash
aws iam create-role \
  --role-name rust-role \
  --assume-role-policy-document \
  '{"Version": "2012-10-17","Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```

Copy the value for `Arn`, which in my case is `arn:aws:iam::124397940292:role/rust-role`, and include it in the next command for the `--iam-role` flag.

```bash
cargo lambda deploy \
  --iam-role FULL_ROLE_ARN \
  --enable-function-url
```

```
🔍 function arn: arn:aws:lambda:us-east-1:124397940292:function:rustrocket
🔗 function url: https://bxzpdr7e3cvanutvreecyvlvfu0grvsk.lambda-url.us-east-1.on.aws/
```

Open the [function URL](https://bxzpdr7e3cvanutvreecyvlvfu0grvsk.lambda-url.us-east-1.on.aws/) to see your function.

![deployed function](https://user-images.githubusercontent.com/12433465/190524714-2235ac63-fd06-439a-9a84-2b5fb069e453.png)

### Send a Request with Invoke Command

Send a request with the [`invoke` subcommand](https://www.cargo-lambda.info/commands/invoke.html) and include a `--data-example` from the [`aws-lambda-events` repository](https://github.com/LegNeato/aws-lambda-events/).

```bash
cargo lambda invoke \
  --remote \
  --data-example apigw-request \
  arn:aws:lambda:us-east-1:124397940292:function:rustrocket
```
