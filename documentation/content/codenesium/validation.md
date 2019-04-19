+++
title = "Validation"
description = ""
weight = 1
alwaysopen = true
+++

We use [Fluent Validation](https://github.com/JeremySkinner/FluentValidation) for our validation needs.

Fluent Validation is an amazing validation library that makes validation fun.

Rules are assigned to models using fluent syntax.

This is a complex example but I want to show how flexible it is and how clear the rules are. 
```
public virtual void AddressLine1Rules()
{
	this.RuleFor(x => x.AddressLine1).NotNull();
	this.RuleFor(x => x).MustAsync(this.BeUniqueGetAddressLine1AddressLine2CityStateProvinceIDPostalCode).When(x => x ?.AddressLine1 != null).WithMessage("Violates unique constraint").WithName(nameof(ApiAddressRequestModel.AddressLine1));
	this.RuleFor(x => x.AddressLine1).Length(0, 60);
}

private async Task<bool> BeUniqueGetAddressLine1AddressLine2CityStateProvinceIDPostalCode(ApiAddressRequestModel model,  CancellationToken cancellationToken)
{
	var record = await this.AddressRepository.GetAddressLine1AddressLine2CityStateProvinceIDPostalCode(model.AddressLine1,model.AddressLine2,model.City,model.StateProvinceID,model.PostalCode);

	return record == null;
}
```

I think the idea is to keep the model validation separate from the service which is what we do. We inject model validators into the services and that helps
us keep single responsibility intact. 

Fluent validation is usually applied using filters on controllers. This works fine but we're trying to keep all of the validation in the service layer.

The initial API just has rules that we can infer from the database like length, nullability and unique constraints. As you apply business rules you can add them to
the existing validators or add new ones separate from the database type validation. I feel like it makes more sense to add your validation to the existing ones
by leveraging the derived validator class to override the generated validators. 

It is also possible, although not recommended, to remove these validators and just let the entity framework validation bubble up errors.  

