<h1>A practical guide to gRPC with</h1>
<img alt="rust logo" src="assets/images/rust.svg" style="width: 300px;" />
<p>Victor Martinez</p>

---

<h2>Contents</h2>

<div class="row">
  <div>

  + Ingredients to build a gRPC API
  + Defining our API with Protocol Buffers
      + Defining a request
      + Defining a response
      + About backwards compatibility
      + Defining a service
  + Generate a Rust library
      + Manual project setup
      + Build script
      + Exposing a library
      + Supporting multiple tonic versions
  + CI/CD
      + The Buf tool
      + Checking for breaking changes
      + How to release

  </div>

  <div>

  + Implementing a gRPC service
    + Implementing the service trait
    + Parsing requests
    + Error handling
  + Building a gRPC server
    + Authentication
    + Tracing
    + Running our server
  + How to deploy
  + How to call our server
    + Creating a gRPC client
    + Authentication
    + Response handling
  + How is it going so far
    + Users
    + Metrics
  </div>
</div>

---

## Ingredients to build a gRPC API

<div class="row">
  <img alt="protocol buffers logo" src="assets/images/protocol-buffers.png" style="width: 700px;" />  <!-- .element: class="fragment" data-fragment-index="1" -->
  <img alt="Buf logo" src="https://cdn.prod.website-files.com/67202403476bad65d88793e7/6720247ef74ab81302be8c70_Buf%20logo%20%5Bcolor%5D.svg" style="width: 300px;" /> <!-- .element: class="fragment" data-fragment-index="2" -->
</div>
<img alt="Tonic logo" src="assets/images/tonic.svg" style="width: 300px" /> <!-- .element: class="fragment" data-fragment-index="3" -->

---

## Defining our API

<img alt="protocol buffers logo" src="assets/images/protocol-buffers.png" style="width: 700px;" />

---

## Project structure

<img alt="protocol buffers logo" src="assets/images/project-structure.png" style="width: 1080px;" />

<a href="https://github.com/primait/es-policy-grpc">https://github.com/primait/es-policy-grpc</a>

---

## Project structure

<img alt="protocol buffers logo" src="assets/images/project-structure-2.png" style="width: 1080px;" />

<a href="https://github.com/primait/es-policy-grpc">https://github.com/primait/es-policy-grpc</a>

---

## Defining a request

```proto
syntax = "proto3";

package es_policy_grpc.messages.issue_policy.request.v1;

import "es_policy_grpc/domain/v1/bundle.proto";
import "es_policy_grpc/domain/v1/coverage.proto";
import "es_policy_grpc/domain/v1/issuing_company.proto";
// ... etc

message IssuePolicyRequest {
  google.protobuf.Timestamp requested_at = 1; 
  google.protobuf.Timestamp start_at = 2;
  google.protobuf.Timestamp end_at = 3;
  google.protobuf.Timestamp purchased_at = 4;
  es_policy_grpc.domain.v1.TransactionInformation transaction = 5;
  es_policy_grpc.domain.v1.IssuingCompany issuing_company = 6;
  string quote_id = 7;
  es_policy_grpc.domain.v1.QuoteSource quote_source = 8;
  string application_id = 9;
  string offer_id = 10;
  es_policy_grpc.domain.v1.Price price = 11;
  es_policy_grpc.domain.v1.Bundle bundle = 12;
  repeated es_policy_grpc.domain.v1.ProductCover covers = 13;
  es_policy_grpc.domain.v1.PolicyHolderInformation policy_holder_information = 14;
  es_policy_grpc.domain.v1.VehicleInformation vehicle_information = 15;
  es_policy_grpc.domain.v1.QuoteVersion quote_version = 16;
} 
```

---

## Defining a request

```proto
syntax = "proto3";

package es_policy_grpc.domain.v1;

import "es_policy_grpc/domain/v1/price.proto";
import "google/protobuf/timestamp.proto";

message ProductCover {
  CoverType cover_type = 1;
  google.protobuf.Timestamp start_at = 2;
  google.protobuf.Timestamp end_at = 3;
  es_policy_grpc.domain.v1.Price price = 4;
  optional RoadsideAssistanceTier roadside_assistance_tier = 5;
}

enum CoverType {
  COVER_TYPE_UNSPECIFIED = 0;
  COVER_TYPE_MANDATORY_THIRD_PARTY_LIABILITY = 1;
  COVER_TYPE_VOLUNTARY_THIRD_PARTY_LIABILITY = 2;
  COVER_TYPE_DRIVER_ACCIDENT = 3;
  COVER_TYPE_WINDSHIELD = 4;
  COVER_TYPE_THEFT = 5;
  // ..etc
}

enum RoadsideAssistanceTier {
  ROADSIDE_ASSISTANCE_TIER_UNSPECIFIED = 0;
  ROADSIDE_ASSISTANCE_TIER_BASE = 1;
  ROADSIDE_ASSISTANCE_TIER_PREMIUM = 2;
  ROADSIDE_ASSISTANCE_TIER_PREMIUM_V2 = 3;
}
```

---

## Defining a response

```proto
syntax = "proto3";

package es_policy_grpc.messages.issue_policy.response.v1;

message IssuePolicyResponse {
  string policy_id = 1;
}
```

---

## About backwards compatibility
