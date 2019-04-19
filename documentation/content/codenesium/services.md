+++
title = "Services"
description = ""
weight = 1
alwaysopen = true
+++

Services are where the business logic lives.

They take models from the Web layer and validate then call repositories to make changes to your database.

The default generated APID doesn't contain business logic for your domain. It only has what the database can tell us.

If your models require other logic then this is where that is implemented. It can be in validators or in methods in the service it's really up to you. 

We have DTOs that the entity framework entities are mapped to and then we map those to request or response models for the API. This may seem redundant but the 
idea is that you're going to want to implement some kind of Aggregate Root type objects at this level and you can use the DTO's in those to implement your custom business logic. 

```
public interface IAddressService
{
	Task<CreateResponse<ApiAddressResponseModel>> Create(
		ApiAddressRequestModel model);

	Task<ActionResponse> Update(int addressID,
								ApiAddressRequestModel model);

	Task<ActionResponse> Delete(int addressID);

	Task<ApiAddressResponseModel> Get(int addressID);

	Task<List<ApiAddressResponseModel>> All(int skip = 0, int take = int.MaxValue, string orderClause = "");

	Task<ApiAddressResponseModel> GetAddressLine1AddressLine2CityStateProvinceIDPostalCode(string addressLine1,string addressLine2,string city,int stateProvinceID,string postalCode);
	Task<List<ApiAddressResponseModel>> GetStateProvinceID(int stateProvinceID);
}
```


##### Mapping
We map entities to DTOs then we map DTOs to response models. You can replace this with [AutoMapper](https://github.com/AutoMapper/AutoMapper).
It's my opinion that you can't use AutoMapper unless you have 100% test coverage where it's going to be used. It makes refactoring difficult because the mapping is done with reflection and leads to runtime bugs when field names change. 
