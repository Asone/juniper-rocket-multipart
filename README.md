# Juniper Rocket Multipart

The current crate provides with a handler that allows using `multipart/data-form` requests on a [Rocket](https://rocket.rs/) webserver with [Juniper](https://github.com/graphql-rust/juniper).

It replicates the default behavior of Juniper for requests with `content-type` header of `application/json` and `application/graphql`. 

## Disclaimer 

The current crate is a still on-going process as it lacks yet :

- Unit tests
- Rebuilding the provided map objects
- Max bytes read per file is still hardcoded

Yet it should be functional. Use it at your own risks. 

## How to use 

You can load the `GraphQLUploadWrapper` the same way you load the GraphQLRequest as both are data guards.
The main difference will be that instead, you'll call the
 execution of the query through the `operations` property
 of the wrapper.

  Below is basic example :

 ```rust
 #[rocket::post("/upload", data = "<request>")]
 pub async fn upload<'r>(
    request: GraphQLUploadWrapper,
    schema: &State<Schema>,
 ) -> GraphQLResponse {
    request.operations.execute(&*schema, &Context).await
}
 ```

 ## Fetching the uploaded files

 In order to fetch the uploaded files
 You'll need to implement your own context object
 That will pass the buffered files to your execution methods.

 Example :
 ```rust
 struct Ctx{
   files: Option<HashMap<String, TempFile>>
 };
 impl juniper::Context for Ctx {}
 ```

 You'll then be able to inject the buffered files to your
 operations like this :
 ```rust
 struct Ctx{ files: Option<HashMap<String, TempFile>> };
 impl juniper::Context for Ctx {}

 #[rocket::post("/upload", data = "<request>")]
 pub async fn upload<'r>(
    request: GraphQLUploadWrapper,
    schema: &State<Schema>,
 ) -> GraphQLResponse {
   request.operations.execute(&*schema, &Ctx{ files: request.files }).await
 }
 ```

 ## Notes about processed files

 The Wrapper does nothing special with the uploaded files aside
 allocating them in heap memory through the Hashmap which means
 they won't be stored anywhere, not even in a temporary folder,
 unless you decide to.

 See [TempFile](./src/temp_file.rs) for more available data and information around uploaded files.