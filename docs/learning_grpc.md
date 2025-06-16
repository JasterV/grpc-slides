# Learning gRPC

Victor Martinez

---

# First, what is RPC?

An idea to extend transfer of control and transmission of data from one machine to another.

<img alt="RPC implementation" src="assets/images/rpc_architecture.png" style="width: 60%;" />

[http://birrell.org/andrew/papers/ImplementingRPC.pdf](http://birrell.org/andrew/papers/ImplementingRPC.pdf)

note:

The concept dates back to 1976 [1]

Building applications that required communicating with a separate machine was difficult and required big expertise

This RPC implementation aimed to make it highly efficient (network-wise) as well as as simple to use as non-remote procedures.

Make it more accessible to build distributed applications

They aimed to provide secure communications with RPC.

Things were shared in plain non secured text.

The program structure would be based in the concept of Stubs.

Five pieces of program are involved when making an RPC call:

User -> User-stub -> RPC communications package (known as RPCRuntime) -> server-stub -> the server.

They auto-generated the client and server stubs:

`The user-stub and server-stub are automatically generated, by a program called Lupine.`

[1] WHITE, J. E. A high-level framework for network-based resource sharing. In Proc. National Computer Conference, (June 1976).

---

<img alt="grpc" src="assets/images/grpc-logo.png" style="width: 600px;" />

*gRPC is a modern open source high performance Remote Procedure Call (RPC) framework that can run in any environment.*

[https://grpc.io/](https://grpc.io/)

note:

google Remote procedure calls

"gRPC was initially created by Google, which has used a single general-purpose RPC infrastructure called **Stubby** to connect the large number of microservices running within and across its data centers. In March 2015, Google decided to build the next version of Stubby and make it open source. The result was **gRPC**"

---

# Why a framework?

gRPC dictates how you will build your network interface.

Code is generated for you batteries included, you must only fill the gaps.

note:

All the underlying details about networking, encoding & more is handled for you.

It is more a framework in the sense of servers. They must use the generated Server Stub, with the only need of implementing the Service interfaces.

Clients will use the generated client Stub. For them the gRPC code will be less intrusive and will feel more like a library

Some implementations wrap the original C library, some don't.

---

## Built on top of HTTP2

So we get for free
- **Multiplexing**
- Header **compression**
- **Server push**
- **TLS**

note:

Explain multiplexing and server push

---

## 4 types of RPC supported

<img alt="rpc types" src="assets/images/rpc_types.svg" style="width: 70%;" />

note:

Explain that each of these RPC types can be specified on the protobuffers IDL

---
## Metadata

Key-value pairs of data used to provide additional information about a call.

Implemented using HTTP/2 headers.

[https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)

note:

gRPC metadata can be sent and received by both the client and the server. Headers are sent from the client to the server before the initial request and from the server to the client before the initial response of an RPC call.

On the link I show, they document the supported values for metadata

Can be useful for: Authentication & tracing

---
## And many more features

- **Health checking** (Service-specific health checking)
- **Interceptors** (Middleware for RPCs)
- **Reflection** (Service discoverability & ease debugging)
- RPC automatic & manual **cancellations**
- Call **retries**
- **Flow control** for streaming
- **Load balancing** (Client requests can be load balanced between multiple servers)

note:

It is important to explain that these features might differ from language to language, since it depends completely on how each of them implements gRPC

- **Flow control** is a mechanism to ensure that a receiver of messages does not get overwhelmed by a fast sender. Flow control prevents data loss, improves performance and increases reliability.

- **Reflection**: Explain that we won't go in detail about reflection but that I believe we should research more about it since it can be useful for better developer experience

- **Health check**: gRPC specifies a standard service API ([health/v1](https://github.com/grpc/grpc-proto/blob/master/grpc/health/v1/health.proto)) for performing health check calls against gRPC servers. An implementation of this service is provided, but you are responsible for updating the health status of your services. It is pluggable, and some languages might not provide it.
---

# Protocol buffers

*Protocol Buffers are language-neutral, platform-neutral extensible mechanisms for serializing structured data.*

https://protobuf.dev/

note:

It is also developed by google.

Explain that it is the default binary serialization format supported by gRPC

Many IDLs have been developed over time. Mozilla, Microsoft, IBM... and more developed their own internal RPC frameworks with their own IDLs [2]

[2] https://en.wikipedia.org/wiki/Interface_description_language

---

## They are a combination of

- The **Interface Definition Language**
- The compiler that **generates code** from IDL files
- Language-specific **runtimes**
- The **serialization format**

note:

Explain what an IDL is

Here we will focus on the IDL and the tooling, we won't focus on the serialization format.

---

## Remarkable features of Protocol buffers

- **Strongly typed** data
- **Language** and **platform neutral**
- **Compact binary format**
- Support for **RPC service definition**
- **Backward** and **Forward compatibility**

note:

Give a short example of why it is backward and forward compatible. Mention tags.

---

## Defining messages

```protobuf
syntax = "proto3";

package decline_renewal.request.v1;

import "google/protobuf/timestamp.proto";

message DeclineRenewalRequest {
  string policy_id = 1;
  google.protobuf.Timestamp requested_at = 2;
  optional string description = 3;
  oneof reason {
    CustomerDeclineRenewalReason customer = 4;
  }
}

enum CustomerDeclineRenewalReason {
  CUSTOMER_DECLINE_RENEWAL_REASON_UNSPECIFIED = 0;
  CUSTOMER_DECLINE_RENEWAL_REASON_COMPETITOR_OFFER = 1;
  CUSTOMER_DECLINE_RENEWAL_REASON_VEHICLE_SOLD = 2;
  CUSTOMER_DECLINE_RENEWAL_REASON_VEHICLE_NOT_PURCHASED = 3;
  CUSTOMER_DECLINE_RENEWAL_REASON_VEHICLE_DEREGISTRATION = 4;
  CUSTOMER_DECLINE_RENEWAL_REASON_NO_INSURANCE_WANTED = 5;
  CUSTOMER_DECLINE_RENEWAL_REASON_DOES_NOT_KNOW = 6;
  CUSTOMER_DECLINE_RENEWAL_REASON_WANTS_GREEN_CARD = 7;
  CUSTOMER_DECLINE_RENEWAL_REASON_INCORRECT_EFFECTIVE_DATE = 8;
  CUSTOMER_DECLINE_RENEWAL_REASON_INCORRECT_PERSONAL_DATA = 9;
  CUSTOMER_DECLINE_RENEWAL_REASON_INCORRECT_DATA_OTHER = 10;
}
```

---

## Defining messages

```protobuf
syntax = "proto3";

package decline_renewal.response.v1;

message DeclineRenewalResponse {
  string policy_id = 1;
}  
```

---

## Defining a service

```protobuf
syntax = "proto3";

package service.v1;

import "es_policy_grpc/messages/amend_termination/request/v1/request.proto";
import "es_policy_grpc/messages/amend_termination/response/v1/response.proto";
// Skipping other imports for the sake of the slide

service PolicyManagementService {
  rpc AmendTermination(amend_termination.request.v1.AmendTerminationRequest)
    returns (amend_termination.response.v1.AmendTerminationResponse);

  rpc TerminatePolicy(terminate_policy.request.v1.TerminatePolicyRequest)
    returns (terminate_policy.response.v1.TerminatePolicyResponse);

  rpc WithdrawPolicy(withdraw_policy.request.v1.WithdrawPolicyRequest)
    returns (withdraw_policy.response.v1.WithdrawPolicyResponse);

  rpc DeclineRenewal(decline_renewal.request.v1.DeclineRenewalRequest)
    returns (decline_renewal.response.v1.DeclineRenewalResponse);
}
```

---
## The protoc compiler

Compiles `.proto` files into code.
Supports plugins for different languages.

```bash
protoc --proto_path=src --python_out=build/gen src/foo.proto
```

note:

`--proto_path` specifies the source directory, `--*_out` the destination directory, and the rest is the path to your `.proto`

---
## Buf CLI

- A **linter** for proto files
- A **formatter** for proto files
- A system to organize your proto files by **workspaces**
- A feature to check for **breaking changes** in your definitions
- A **plugin system** to compile proto files into multiple formats
- **Editor integration**
- And more!

[https://buf.build/product/cli](https://buf.build/product/cli)

note:

Explain that it builds on top of protoc. Be very short here, just mention the tool briefly. It is important because we use it.

---

# gRPC in the Rust ecosystem


<img alt="grpc" src="assets/images/grpc-logo.png" style="width: 300px;" />

:heart:


<img alt="rust logo" src="assets/images/rust.svg" style="width: 300px;" />

---
# Tower

<img alt="tower" src="assets/images/tower.png" style="width: 200px;" />

note:

Tower is a library of modular and reusable components for building robust networking clients and servers.

Tonic is built on top of Tower

It's core abstraction is the Service, which we see in the next slide.

It exposes already a set of basic reusable services to solve common networking patterns such as timeouts and rate limiting.

---
## Tower service

```rust
pub trait Service<Request> {
    type Response;
    type Error;
    type Future: Future<Output = Result<Self::Response, Self::Error>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>>;
  
    fn call(&mut self, req: Request) -> Self::Future;
}
```

note:

Tower’s fundamental abstraction.

An asynchronous function from a `Request` to a `Response`.

The `Service` trait is a simplified interface making it easy to write network applications in a modular and reusable way, decoupled from the underlying protocol.

It immediately returns a `Future` representing the eventual completion of processing the request. 

The processing may depend on calling other services. At some point in the future, the processing will complete, and the `Future` will resolve to a response or error.

---
## Layers

```rust
pub trait Layer<S> {
    type Service;
    
    fn layer(&self, inner: S) -> Self::Service;
}
```

note:

Mechanism to layer services. It allows us to wrap a generic service with another one. It can be used to wrap a reusable service which is meant to act as a middleware around another service.

---
## Building a layered service

```rust
ServiceBuilder::new()
    .timeout(Duration::from_secs(10))
    .layer(OpenTelemetryServerTracingLayer::new_for_grpc())
    .layer(JwtAuthLayer::new(jwks_client, "starsky"))
    .named_layer(StarskyServer::new(starsky_service));
```

note:

A real example of a layered service from Starsky. Slightly simplified for the sake of the presentation.
The flow will be the following: 
Timeout -> SSRHL -> Tracing -> SSRHL -> Auth -> Starsky service

---

## Building a layered service

<img alt="tower" src="assets/images/layers-diagram.svg" style="max-width: 35%;" />

---
# Tonic

<img alt="tonic logo" src="assets/images/tonic.svg" style="width: 200px;" />

*A gRPC over HTTP/2 rust implementation focused on high performance, interoperability, and flexibility*

[https://github.com/hyperium/tonic](https://github.com/hyperium/tonic)

note:

It has first class support for async/await.

The main goal of tonic is to provide a generic gRPC implementation over HTTP/2 framing. 

Codegen tools need to be used to generate the client and server stubs that will encode and decode the binary data and deal with other gRPC features such as streaming.

---
## Features

- **Health check** of services
- **Interceptors** 
- **Reflection**
- **Code generation** from proto definitions
- RPC cancellation via **timeouts**
- Bidirectional **streaming**
- **Load balancing**
- Request/Response **compression**
- **TLS**
- Extensible via **Tower** services

note:

These are only a few notable features, it provides more for sure

---
## Generating code from Proto definitions :gear:

```rust
// build.rs
let mut prost_build = prost_build::Config::new();
prost_build.protoc_arg("--experimental_allow_proto3_optional");
prost_build.compile_protos(
    &[
        "proto/es_policy_grpc/messages/terminate_policy/request/v1/request.proto",
        "proto/es_policy_grpc/messages/terminate_policy/response/v1/response.proto",
        "proto/es_policy_grpc/messages/withdraw_policy/request/v1/request.proto",
        "proto/es_policy_grpc/messages/withdraw_policy/response/v1/response.proto",
        "proto/es_policy_grpc/messages/decline_renewal/request/v1/request.proto",
        "proto/es_policy_grpc/messages/decline_renewal/response/v1/response.proto",
        "proto/es_policy_grpc/messages/amend_termination/request/v1/request.proto",
        "proto/es_policy_grpc/messages/amend_termination/response/v1/response.proto",
    ],
    &["proto"],
)?;

tonic_build::configure()
    .protoc_arg("--experimental_allow_proto3_optional")
    .compile_protos(
        &["proto/es_policy_grpc/service/v1/service.proto"],
        &["proto"],
    )
    .unwrap();
```

note:

First we need to talk about how do we generate code from our protobuf definitions.

---
## Expose the generated code as a library

```rust
// lib.rs

pub mod messages {
    pub mod decline_renewal {
        pub mod request {
            pub mod v1 {
                include!(concat!(env!("OUT_DIR"), "/es_policy_grpc.messages.decline_renewal.request.v1.rs"));
            }
        }

        pub mod response {
            pub mod v1 {
                include!(concat!(env!("OUT_DIR"), "/es_policy_grpc.messages.decline_renewal.response.v1.rs"));
            }
        }
    }
    // ..
}

pub mod policy_service {
    pub mod v1 {
        include!(concat!(env!("OUT_DIR"), "/es_policy_grpc.service.v1.rs"));
    }
}
```

note:

We need to expose the generated code through our lib.rs

---

## Filling the gaps

```rust
pub trait PolicyManagementService {
    async fn decline_renewal(
        &self,
        request: Request<DeclineRenewalRequest>,
    ) -> Result<Response<DeclineRenewalResponse>, Status>
	// ...
}
```

note:

We get a trait generated from the Protobuf Service definition

---

## Filling the gaps

```rust
use es_policy_grpc::policy_service::v1::PolicyManagementService;
use es_policy_grpc::messages::decline_renewal::request::v1::DeclineRenewalyRequest;
use es_policy_grpc::messages::decline_renewal::response::v1::DeclineRenewalResponse;
use tonic::{Request, Response, Status};

pub struct PolicyManagementServiceImpl {
    application: Arc<dyn ApplicationServices>,
}

impl PolicyManagementService for PolicyManagementServiceImpl {
    async fn decline_renewal(
        &self,
        request: Request<DeclineRenewalRequest>,
    ) -> Result<Response<DeclineRenewalResponse>, Status> {
        let request = request.into_inner();

        let policy_id = Uuid::parse_str(&request.policy_id).unwrap();

        let policy = self.application.find_policy(policy_id).await.unwrap();

        let details: TerminateDetails = request.try_to_domain(policy.expiration_date()).unwrap()

        self.application.cancel_policy(policy_id.into(), details).await;

        Ok(Response::new(DeclineRenewalResponse {
            policy_id: policy_id.to_string(),
        }))
    }
    // ..
}

```

---
## Building the server

```rust
use tonic::Server as GrpcServer;
use es_policy_grpc::policy_service::v1::PolicyManagementServiceServer as PolicyManagementServerStub;

let server = 
	// gRPC server provided by Tonic
	GrpcServer::builder() 
		.add_service(
			// Generated Policy Management Server Stub
			PolicyManagementServerStub::new(
				// Implementation of the service
				PolicyManagementServiceImpl::new(application)
			) 
	);

let listener = TcpListener::bind(("0.0.0.0", grpc_port)).await?;

server.serve(listener).await?;
```

note:

Simple build of a Tonic Server. We will dive into how to add middleware later.

Highlight the fact that at the end of the day the gRPC server will be listening to a TCP port like any other HTTP2 server.

---
## Building the client

```rust
use es_policy_grpc::policy_service::v1::PolicyManagementServiceClient as PolicyManagementClientStub;
use tonic::{metadata::MetadataValue, Request};
use es_policy_grpc::messages::decline_renewal::request::v1::{
    DeclineRenewalRequest,
    DeclineRenewalReason,
    CustomerDeclineRenewalReason
};

// Auto-generated client stub
let mut client = PolicyManagementClientStub::connect("http://localhost:50051").await?;

let mut request = Request::new(DeclineRenewalRequest {
    policy_id: uuid::Uuid::new_v4(),
    requested_at: DateTime::now(),
    description: Some("dummy".into()),
    reason: DeclineRenewalReason::Customer(
        CustomerDeclineRenewalReason::VehicleSold
    )
});

let token: MetadataValue<_> = "Bearer some-auth-token".parse()?;

request.metadata_mut.insert(http::AUTHORIZATION, token);

let _response = client.generate_contract(request).await?;
```

note:

What if we wanted to add those headers for every request? Now we talk about interceptors

---

## Health checking gRPC services

Tonic provides a health check service implementing a standard gRPC health checking protocol.

[https://github.com/grpc/grpc/blob/master/doc/health-checking.md](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)

note:

A GRPC service is used as the health checking mechanism. 

Since it is a GRPC service itself, doing a health check is in the same format as a normal rpc. 

It has rich semantics such as per-service health status. 

The server has full control over the access of the health checking service.

---
## Health service definition

```protobuf
syntax = "proto3";

package grpc.health.v1;

message HealthCheckRequest {
  string service = 1;
}

message HealthCheckResponse {
  enum ServingStatus {
    UNKNOWN = 0;
    SERVING = 1;
    NOT_SERVING = 2;
    SERVICE_UNKNOWN = 3; // Used only by the Watch method.
  }
  ServingStatus status = 1;
}

service Health {
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);
  rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
}
```

This definition is provided by the official gRPC docs, each language runtime might implement it or not.

[https://github.com/grpc/grpc/blob/master/doc/health-checking.md](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)
---
## Enabling the health service

```rust
use es_policy_grpc::policy_service::v1::PolicyManagementServiceServer as PolicyManagementServerStub;
use tonic_health::server::health_reporter;
use tonic::Server as GrpcServer;

let (health_reporter, health_service) = health_reporter();

health_reporter
    .set_serving::<PolicyManagementServerStub<PolicyManagementServiceImpl>>()
    .await;

GrpcServer::builder()
	// Add other layers
	.layer(..)
	.add_service(health_service)
	.serve(addr)
	.await?;
```

note:

Make it clear that we are using the `tonic-health` crate which doesn't come by default with `tonic`.

---

# Building middleware with Tower

So, how can we take advantage of Tower in gRPC?

---

## Authorization middleware

Auth0 M2M authorization

---
## Authentication service

```rust
// Tower Service used as a JWT Auth middleware.
pub struct JwtAuth<T: JwtDecoder, S> {
    jwt_decoder: Arc<T>,
    audience: String,
    inner: S,
}
```

note:

First we build a struct that will contain a generic inner service protected by our auth service.

The audience represents the audience set in Auth0, which is our API identifier.

The JwksClient contains the public keys to verify the signature of incoming tokens.

---

## Authentication service

```rust
impl<T: JwtDecoder, S> JwtAuth<T, S> {
    async fn authorize<Req, Res>(&self, req: http::Request<Req>) -> Result<http::Request<Req>, http::Response<Res>>
    where
        Res: Default,
    {
        let token = req.headers()
            .get(http::AUTHORIZATION)
            .ok_or_else(make_unauthorized_response)?
            .strip_prefix("Bearer ")
            .ok_or_else(make_unauthorized_response)?

        if let Err(_err) = self.jwt_decoder.decode::<serde_json::Value>(token, &self.audience).await {
            return Err(make_unauthorized_response());
        }

        Ok(req)
    }
}
```

note:

Here we implement the authentication logic, we are not implementing yet the service trait.

I've simplified the code for the sake of the slide.

We assume that `make_unauthorized_response` will build a gRPC unauthorized response.

---

## Authentication service

```rust
use std::task::{Context, Poll};
use http::{Request, Response};

impl<Req, Res, S, T> Service<Request<Req>> for JwtAuth<T, S>
where
    S: Service<Request<Req>, Response = Response<Res>>,
    T: JwtDecoder,
    // .. Skipping other constraints
{
    type Response = S::Response;
    type Error = S::Error;
    type Future = BoxFuture<'static, Result<Self::Response, Self::Error>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, req: Request<Req>) -> Self::Future {
        let mut this = self.clone();

        async move {
            match this.authorize(req).await {
                Ok(req) => this.inner.call(req).await,
                Err(res) => Ok(res),
            }
        }
        .boxed()
    }
}
```

note:

Note the use of `async move` inside `call` given that `call` is not defined as an async function on the trait definition

---

## Authentication layer

```rust
// Reusable Tower Layer meant to wrap
// a JWT Auth middleware Service around a generic service
pub struct JwtAuthLayer<T: JwtDecoder> {
    jwt_decoder: Arc<T>,
    audience: String,
}

impl<T: JwtDecoder> JwtAuthLayer<T> {
    pub fn new(jwt_decoder: T, audience: impl Into<String>) -> Self {
        Self {
            jwt_decoder: Arc::new(jwt_decoder),
            audience: audience.into(),
        }
    }
}
```

note:

Although confusing, the purpose of the layer is to make the usage of the middleware more user-friendly

---

## Authentication layer

```rust
impl<T, S> Layer<S> for JwtAuthLayer<T>
where
    T: JwtDecoder,
{
    type Service = JwtAuth<T, S>;

    fn layer(&self, inner: S) -> Self::Service {
        JwtAuth {
            jwt_decoder: self.jwt_decoder.clone(),
            audience: self.audience.clone(),
            inner,
        }
    }
}
```

note:

What is done inside of the `layer` function could just be done manually, but it is done here for better user experience later.

---
## Attaching it to our gRPC server

```rust
use es_policy_grpc::policy_service::v1::PolicyManagementServiceServer as PolicyManagementServerStub;
use tonic::Server as GrpcServer;
use tower::ServiceBuilder;

// ...

let authenticated_apis = ServiceBuilder::new()
    .layer(JwtAuthLayer::new(jwks_client, AUDIENCE))
    .service(PolicyManagementServerStub::new(
        PolicyManagementServiceImpl::new(application),
    ));

let server = GrpcServer::builder().add_service(authenticated_apis);
```

note:

Simplified version of our real server implementation in `es-be`

---
## Tracing Layer

Let's build another Tower service.

Interceptors are not the best fit, we want to trace responses too.

---

## Building a span from a request

```rust
fn make_span<B>(request: &http::Request<B>) -> tracing::Span {
    // We'll assume server_info() works
    let ServerInfo { host, port, .. } = server_info(request);

    let mut headers = request.headers();

    let name = request.uri().path().trim_start_matches('/');

    let (service, method) = name
        .split_once('/')
        .expect("gRPC paths should be formatted as $service/$method");

    tracing::info_span!(
        "gRPC request",
        otel.name = %name,
        rpc.grpc.request.metadata = ?headers,
        rpc.method = method,
        rpc.service = service,
        rpc.system = "grpc",
        server.address = %host,
        server.port = port,
        span.kind = "server",
        // set by the response span
        otel.status_code = tracing::field::Empty,
        rpc.grpc.response.metadata = tracing::field::Empty,
        rpc.grpc.status_code = tracing::field::Empty,
    )
}
```

note:

Explain how this is a simplified version of the real implementation in `prima_tower`

---

## Updating the span with the response

```rust
fn on_response<B>(response: &http::Response<B>, span: &tracing::Span) {
    let mut headers = response.headers().clone();
    redact_sensitive_headers(&mut headers);

    let code = tonic::Status::from_header_map(&headers)
        .map(|status| status.code())
        .unwrap_or(tonic::Code::Ok);

    span.record("rpc.grpc.status_code", code as i32);
    span.record("grpc.response.header", format!("{:?}", headers));

    if matches!(
        code,
        tonic::Code::Unknown
            | tonic::Code::DeadlineExceeded
            | tonic::Code::Unimplemented
            | tonic::Code::Internal
            | tonic::Code::Unavailable
            | tonic::Code::DataLoss
    ) {
        span.record("otel.status_code", "ERROR");
    }
}
```

note:

We will see in a second how the span we receive by parameters is the same span we created when handling the request

---

## Tracing service

```rust
// Tower Service acting as a Tracing middleware
// for gRPC requests and responses
pub struct OpenTelemetryTracer<S> {
    inner: S,
}
```

note:

We need to implement the service that will act as the tracing middleware

---

## Tracing service

```rust
use std::task::{Context, Poll};
use http::{Request, Response};
use opentelemetry_http::HeaderExtractor;
use opentelemetry_sdk::propagation::TraceContextPropagator;

impl<Req, Res, S> Service<Request<Req>> for OpenTelemetryTracer<S>
where
    S: Service<Request<Req>, Response = Response<Res>>,
    S::Future: Send + 'static,
{
    type Response = S::Response;
    type Error = S::Error;
    type Future = BoxFuture<'static, Result<Self::Response, Self::Error>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, req: Request<Req>) -> Self::Future {
        let parent_context = TraceContextPropagator::new().extract(&HeaderExtractor(req.headers()));

        let span = make_span(&req);
        span.set_parent(parent_context);

        self.inner.call(req).instrument(span.clone()).inspect_ok(move |response| {
            on_response(response, &span);
        })
        .boxed()
    }
}
```

note:

Again, the code is simplified for the slides purpose.

Note how the same span is used to track the request and response.

Then that span is used as the parent span for the inner service call.

---
## Tracing layer

```rust
pub struct OpenTelemetryTracingLayer {}

impl OpenTelemetryTracingLayer {
    pub fn new() -> Self {
        Self {}
    }
}
```

note:

As we've mentioned, layers exist for better development experience, services could be layered manually.

In this case we don't need any data to be added to the layer.

---
## Tracing layer

```rust
impl<S> Layer<S> for OpenTelemetryTracingLayer {
    type Service = OpenTelemetryTracer<S>;

    fn layer(&self, inner: S) -> Self::Service {
        OpenTelemetryServerTracing { inner }
    }
}
```

---
## Attaching it to our gRPC server

```rust
use tonic::Server as GrpcServer;

GrpcServer::builder()
    .layer(OpenTelemetryTracingLayer::new())
    // layer other services to benefit from tracing
    .serve(addr)
    .await?;   
```

note:

It is this simple :)

---

# Thank you for your time 

:heart:
