+++
title = "Controllers"
description = ""
weight = 1
alwaysopen = true
+++

We support the standard REST methods and also generate methods for database constraints and foreign keys. Many to many relationships through a reference table can be 
set as a reference table in the API settings which will automatically create methods on the two reference tables. 

```
[HttpGet]
[Route("")]
[ReadOnly]
[ProducesResponseType(typeof(List<ApiDeviceResponseModel>), 200)]
public async virtual Task<IActionResult> All(int? limit, int? offset)

[HttpGet]
[Route("{id}")]
[ReadOnly]
[ProducesResponseType(typeof(ApiDeviceResponseModel), 200)]
[ProducesResponseType(typeof(void), 404)]
public async virtual Task<IActionResult> Get(int id)

[HttpPost]
[Route("BulkInsert")]
[UnitOfWork]
[ProducesResponseType(typeof(List<ApiDeviceResponseModel>), 200)]
[ProducesResponseType(typeof(void), 413)]
[ProducesResponseType(typeof(ActionResponse), 422)]
public virtual async Task<IActionResult> BulkInsert([FromBody] List<ApiDeviceRequestModel> models)

[HttpPost]
[Route("")]
[UnitOfWork]
[ProducesResponseType(typeof(ApiDeviceResponseModel), 201)]
[ProducesResponseType(typeof(CreateResponse<int>), 422)]
public virtual async Task<IActionResult> Create([FromBody] ApiDeviceRequestModel model)

[HttpPatch]
[Route("{id}")]
[UnitOfWork]
[ProducesResponseType(typeof(ApiDeviceResponseModel), 200)]
[ProducesResponseType(typeof(void), 404)]
[ProducesResponseType(typeof(ActionResponse), 422)]
public virtual async Task<IActionResult> Patch(int id, [FromBody] JsonPatchDocument<ApiDeviceRequestModel> patch)

[HttpPut]
[Route("{id}")]
[UnitOfWork]
[ProducesResponseType(typeof(ApiDeviceResponseModel), 200)]
[ProducesResponseType(typeof(void), 404)]
[ProducesResponseType(typeof(ActionResponse), 422)]
public virtual async Task<IActionResult> Update(int id, [FromBody] ApiDeviceRequestModel model)

[HttpDelete]
[Route("{id}")]
[UnitOfWork]
[ProducesResponseType(typeof(void), 204)]
[ProducesResponseType(typeof(ActionResponse), 422)]
public virtual async Task<IActionResult> Delete(int id)

byPublicId is a unique index on table device. We also generate methods to get return key records to this table. 
[HttpGet]
[Route("byPublicId/{publicId}")]
[ReadOnly]
[ProducesResponseType(typeof(ApiDeviceResponseModel), 200)]
[ProducesResponseType(typeof(void), 404)]
public async virtual Task<IActionResult> ByPublicId(Guid publicId)
```