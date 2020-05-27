
# Result and Result<> object

## What is it?
Result object is my n-th iteration of global return object handling. Instead of returning the type itself, you can return the object wrapped inside the Result<>, or return just Result object with ResultType or ErrorDescription.

You can also specify custom ErrorTypes or ResultTypes, throw it as Exception or use the Result as global WebApi response.

## Why should I use it?
### Without the Result
Let's say that you have this simple class:

    public Customer GetCustomerById(int id)
        {
          try
          {
            Customer customer = dataStore.GetCustomer(id);
    
            if (customer == null || !customer.IsPublic)
              return null;
          
            return customer;
          }
          catch (Exception e)
          {
            // TODO Log the error
            return null;
          }
        }

And you will call it like this:

    Customer customer = GetCustomerById(42);

Now you can check the return if it is null or not. But you will never know the reason why it is null. 
- Is it because the customer was not found?
- Is it because the customer is not public?
- Is it because some network error?

### With the Result
Now consider the following method using Result.

    public Result<Customer> GetCustomerByIdWithResult(int id)
        {
          try
          {
            Customer customer = dataStore.GetCustomer(id);
    
            if(customer == null)
              return new Result<Customer>(ErrorType.NotFound);
    
            if (!customer.IsPublic)
              return new Result<Customer>(ErrorType.NotAuthorized, "Customer is not public yet.");
          
            return new Result<Customer>(ResultType.Ok).SetOutput(customer);
          }
          catch (Exception e)
          {
            // TODO Log the error
    
            return new Result<Customer>(ErrorType.NetworkError, "Error while loading the customer from DB.", e);
          }
        }

Now the return type of the method has changed and allowed us to extend the method a bit.

    // With usage of Result
    Result<Customer> customerResult = GetCustomerByIdWithResult(42);
    
    if (customerResult.Conclusion)
    {
      // Do fun stuff. Your Customer entity is accessible like this:
      Customer customer = customerResult.Output;
    }
    else
    {
      // Handle the problem. You can access full error description.
      ErrorDescription errorDescription = customerResult.ErrorDescription;
    
      // You can also check if it is some specific error. If so, you can throw it as Exception.
      if (errorDescription.ErrorType == ErrorType.NetworkError)
        throw errorDescription.AsException();
    
      // Or just show the error message to the user
      Console.WriteLine(errorDescription.ErrorMessage);
    
      // Or you can check the original Exception (if any)
      if (errorDescription.Exception != null && errorDescription.Exception is ArgumentException)
      {
        // Do some other stuff
      }
    
    }


Do you see the benefit there? Also, the Result obejct has non-generic base type. So in case you have simple command like MakeCustomerPublic(), your return type can be just **Result without any output type**.

# How will I integrate it?
Just **download the Result_AllInOneFile.cs** and do whatever you want with it. This structure is just for publication. You will probably split all classes into separate files, update namespaces and extend ResultTypes or ErrorTypes.
