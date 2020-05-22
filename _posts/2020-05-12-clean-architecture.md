---
layout: post
title:  "Clean Architecture in .NET Core"
date:   2020-05-12 10:21:09 +0100
author: James Eastham
categories: dotnet architecture software
---
I recently wrote an article around the importance of automating the coding standards around your codebase.

Whether that be tabs vs spaces, the number of classes per file or the way comments are structured. It makes the legibility of code so much better when there is a standard and common theme.

The standard repo I use can be found [here](https://github.com/jeastham1993/standard-repo). I've now gone a step further with these standards and added my understanding of the Clean Architecture principles.

# Clean Architecture

Clean Architecture is a way of designing and building software first proposed by Uncle Bob Martin in his book of the same name.

![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/34fsbx5oopko835yl8bj.jpg)

This image has been repeated many times around the web (all credit to Uncle Bob) but it really clearly defines the way an application should be designed.

Each circle should only know anything about the circles within, no dependencies should extend outwards.

So what does that mean for our software? *All the following examples can be found in my GitHub repo*

## Entities

Entities are the purest representation of a business domain. In my example application for managing loans for a local bank, the entities are as follows.

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using CleanArchitecture.Entities.Exceptions;

namespace CleanArchitecture.Core.Entities
{
    /// <summary>
    /// Encapsulates all code for managing loans.
    /// </summary>
    public abstract class Loan
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="Loan"/> class.
        /// </summary>
        /// <param name="principal">The principal valie of the loan.</param>
        /// <param name="rate">The annual interest rate percentage.</param>
        /// <param name="period">The term of the loan.</param>
        internal Loan(decimal principal, decimal rate, decimal period)
        {
            this.Principal = principal;
            this.Rate = rate;
            this.Period = period;
            this.Balance = principal;
        }

        /// <summary>
        /// Gets the standard late fee to be charged.
        /// </summary>
        public virtual decimal LateFee
        {
            get
            {
                return 125M;
            }
        }

        /// <summary>
        /// Gets the initial principal of the loan.
        /// </summary>
        public decimal Principal { get; private set; }

        /// <summary>
        /// Gets the interest rate of the loan.
        /// </summary>
        public decimal Rate { get; private set; }

        /// <summary>
        /// Gets the period the loan will be repayed over.
        /// </summary>
        public decimal Period { get; private set; }

        /// <summary>
        /// Gets the current balance of the loan.
        /// </summary>
        public decimal Balance { get; private set; }

        /// <summary>
        /// Make a payment against the loan.
        /// </summary>
        /// <param name="paymentAmount">The value of the payment made.</param>
        public virtual void MakePayment(decimal paymentAmount)
        {
            var newCalculatedBalance = this.Balance - paymentAmount;

            if (newCalculatedBalance < 0)
            {
                throw new LoanOverpaymentException($"A payment of {paymentAmount} would take the current loan balance below 0");
            }

            this.Balance = this.Balance - paymentAmount;
        }

        /// <summary>
        /// Apply the interest for the elapsed period.
        /// </summary>
        /// <returns>The total accrued interest value.</returns>
        public virtual decimal ApplyInterest()
        {
            this.Balance = this.Principal * (1 + (this.Rate / 100));

            return this.Balance - this.Principal;
        }

        /// <summary>
        /// Charge a late payment fee to this loan.
        /// </summary>
        /// <returns>The new balance after the late fee has been added.</returns>
        public virtual decimal ChargeLateFee()
        {
            this.Balance = this.Balance + this.LateFee;

            return this.Balance;
        }
    }
}
```

A loan is the most fundamental entity that the bank department works with. Within each loan:

- A payment can be made off the balance
- Interest can be applied
- A late fee can be charged

Following the Open/Closed Principle, the class is created as abstract to allow different types of loans to be created. An example can be seen in the concrete BasicLoan class

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace CleanArchitecture.Core.Entities
{
    /// <summary>
    /// A basic loan implementation.
    /// </summary>
    public class BasicLoan : Loan
    {
        internal BasicLoan(decimal principal, decimal rate, decimal term)
            : base(principal, rate, term)
        {
        }
    }
}
```

A basic loan doesn't override any of the base Loan functionality but in the future, it may well be extended.

The bank also has Customers. Again these are represented in a way that is as aligned as possible to the actual business use case. Using the same terminology as the business uses for the entities makes communication between developers and domain experts extremely easy. 

```csharp
using System;
using System.Collections.Generic;
using System.Runtime.CompilerServices;
using System.Text;

[assembly: InternalsVisibleTo("CleanArchitecture.UnitTest")]

namespace CleanArchitecture.Core.Entities
{
    /// <summary>
    /// Encapsulates all logic for a customer entity.
    /// </summary>
    public class Customer
    {
        private int _age;

        /// <summary>
        /// Initializes a new instance of the <see cref="Customer"/> class.
        /// </summary>
        /// <param name="name">The new customers name.</param>
        /// <param name="address">The new customers address.</param>
        /// <param name="dateOfBirth">The new customers date of birth.</param>
        /// <param name="nationalInsuranceNumber">The new customers national insurance number.</param>
        internal Customer(string name, string address, DateTime dateOfBirth, string nationalInsuranceNumber)
        {
            this.CustomerId = Guid.NewGuid().ToString();
            this.Name = name;
            this.Address = address;
            this.DateOfBirth = dateOfBirth;
            this.NationalInsuranceNumber = nationalInsuranceNumber;
        }

        private Customer()
        {
        }

        /// <summary>
        /// Gets the internal identifier of this customer.
        /// </summary>
        public string CustomerId { get; private set; }

        /// <summary>
        /// Gets the name of the customer.
        /// </summary>
        public string Name { get; private set; }

        /// <summary>
        /// Gets the address of the customer.
        /// </summary>
        public string Address { get; private set; }

        /// <summary>
        /// Gets the customers date of birth.
        /// </summary>
        public DateTime DateOfBirth { get; private set; }

        /// <summary>
        /// Gets the national insurance number of the customer.
        /// </summary>
        public string NationalInsuranceNumber { get; private set; }

        /// <summary>
        /// Gets the customers credit score.
        /// </summary>
        public decimal CreditScore { get; private set; }

        /// <summary>
        /// Gets the age of the person based on their <see cref="Customer.DateOfBirth"/>.
        /// </summary>
        public int Age
        {
            get
            {
                if (this._age <= 0)
                {
                    this._age = new DateTime(DateTime.Now.Subtract(this.DateOfBirth).Ticks).Year - 1;
                }

                return this._age;
            }
        }

        internal void UpdateCreditScore(decimal newCreditScore)
        {
            if (newCreditScore != this.CreditScore)
            {
                this.CreditScore = newCreditScore;
            }
        }
    }
}
```

Within the Entities 'circle' I also hold the interface that allows interactions with the customer database.

One of the biggest takeaways from Clean Architecture is to keep the business logic and use cases as far away from the details (dB provider, user interactions) as possible.

It's for this reason, the Entities have a concept of a ICustomers datastore but no actual understanding or care about how the Customers datastore actually works. 

{% highlight csharp linenos %}
```csharp
// ------------------------------------------------------------
// Copyright (c) James Eastham.
// ------------------------------------------------------------

using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;

namespace CleanArchitecture.Core.Entities
{
    /// <summary>
    /// Encapsulates all persistance of customer records.
    /// </summary>
    public interface ICustomerDatabase
    {
        /// <summary>
        /// Store the customer object in the database.
        /// </summary>
        /// <param name="customer">The <see cref="Customer"/> to be stored.</param>
        public void Store(Customer customer);
    }
}
```
{% endhighlight %}

Once we have the Entities mapped out, we can then move on to the next circle outwards. 

## Use Cases

Use cases hold an actual business use of the application. If the entities hold the core business objects, the use cases hold the logic on how these objects work together.

Uncle Bob puts it quite nicely (paraphrasing here)

"Using a bank as an example. The calculation of the interest on a loan would come under an entity, as that is a rule that is fundamental to the business. It is a critical business rule.

However, a system that determines if a specific customer should be allowed to take out a loan or not is something that would benefit from being automated. That is a use case. A use case is a description of the way an automated system is used, it specifies the inputs to be provided by the user and the outputs returned to the user."

Keeping that exact sample in mind, let's add a use case for checking to see if a customer can take out a loan.

First, we will look at the expected inputs and outputs.

From the conversation we had with a domain expert at the bank, we know there are some key decisions that govern if a loan is accepted or not.

- The customer must have a valid name, address and National Insurance number
- They must be older than 18
- They must have a credit score greater than 500

Translating that logic into request/response objects. 

```csharp
// ------------------------------------------------------------
// Copyright (c) James Eastham.
// ------------------------------------------------------------

using System;

namespace CleanArchitecture.Core.Requests
{
    /// <summary>
    /// Gather required data for a new loan.
    /// </summary>
    protected class GatherContactInfoRequest
        : IRequest<GatherContactInfoResponse>
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="GatherContactInfoRequest"/> class.
        /// </summary>
        /// <param name="name">The applicants name.</param>
        /// <param name="address">The applicants address.</param>
        /// <param name="dateOfBirth">The applicants date of birth.</param>
        /// <param name="nationalInsuranceNumber">The applicants NI number.</param>
        public GatherContactInfoRequest(string name, string address, DateTime dateOfBirth, string nationalInsuranceNumber)
        {
            this.Name = name;
            this.Address = address;
            this.DateOfBirth = dateOfBirth;
            this.NationalInsuranceNumber = nationalInsuranceNumber;
        }

        /// <summary>
        /// Gets the name.
        /// </summary>
        public string Name { get; private set; }

        /// <summary>
        /// Gets the address.
        /// </summary>
        public string Address { get; private set; }

        /// <summary>
        /// Gets the date of birth.
        /// </summary>
        public DateTime DateOfBirth { get; private set; }

        /// <summary>
        /// Gets the National Insurance number.
        /// </summary>
        public string NationalInsuranceNumber { get; private set; }
    }
}
```

```csharp
// ------------------------------------------------------------
// Copyright (c) James Eastham.
// ------------------------------------------------------------

using System;
using System.Collections.Generic;

namespace CleanArchitecture.Core.Requests
{
    /// <summary>
    /// Response from a successul gather of ContractInfo.
    /// </summary>
    protected class GatherContactInfoResponse
        : BaseResponse
    {
        internal GatherContactInfoResponse(string name, string address, DateTime dateOfBirth, string nationalInsuranceNumber)
            : base()
        {
            this.Name = name;
            this.Address = address;
            this.DateOfBirth = dateOfBirth;
            this.NationalInsuranceNumber = nationalInsuranceNumber;
        }

        /// <summary>
        /// Gets or sets the applicants name.
        /// </summary>
        public string Name { get; set; }

        /// <summary>
        /// Gets or sets the applicants address.
        /// </summary>
        public string Address { get; set; }

        /// <summary>
        /// Gets or sets the applicants date of birth.
        /// </summary>
        public DateTime DateOfBirth { get; set; }

        /// <summary>
        /// Gets or sets the applicants national insurance number.
        /// </summary>
        public string NationalInsuranceNumber { get; set; }

        /// <summary>
        /// Gets or sets the applicants credit score.
        /// </summary>
        public decimal CreditScore { get; set; }

        /// <summary>
        /// Gets a value indicating if the customer has been accepted for a loan.
        /// </summary>
        public bool IsAcceptedForLoan  { get; private set; }
    }
}
```

We end up with the two above classes. Name, address, date of birth and NI number go in. A credit score and 'IsAcceptedForLoan' comes out.

Keeping the request and response in mind, and using the recommened best Clean Architecture practice, we can create a GatherContactInfoInteractor.

```csharp
// ------------------------------------------------------------
// Copyright (c) James Eastham.
// ------------------------------------------------------------

using System;
using System.Threading.Tasks;
using CleanArchitecture.Core.Entities;
using CleanArchitecture.Core.Services;

namespace CleanArchitecture.Core.Requests
{
    /// <summary>
    /// Handles a <see cref="GatherContactInfoRequest"/>.
    /// </summary>
    public class GatherContactInfoInteractor
        : IRequestHandler<GatherContactInfoRequest, GatherContactInfoResponse>
    {
        private readonly ICreditScoreService _creditScoreService;
        private readonly ICustomerDatabase _customerDatabase;

        /// <summary>
        /// Initializes a new instance of the <see cref="GatherContactInfoInteractor"/> class.
        /// </summary>
        /// <param name="creditScoreService">A <see cref="ICreditScoreService"/>.</param>
        /// <param name="customerDatabase">A <see cref="ICustomerDatabase"/>.</param>
        public GatherContactInfoInteractor(ICreditScoreService creditScoreService, ICustomerDatabase customerDatabase)
        {
            this._creditScoreService = creditScoreService;
            this._customerDatabase = customerDatabase;
        }

        /// <summary>
        /// Handle the given <see cref="GatherContactInfoRequest"/>.
        /// </summary>
        /// <param name="request">A <see cref="GatherContactInfoRequest"/>.</param>
        /// <returns>A <see cref="GatherContactInfoResponse"/>.</returns>
        public GatherContactInfoResponse Handle(GatherContactInfoRequest request)
        {
            if (request is null)
            {
                throw new ArgumentNullException(nameof(request));
            }

            var response = new GatherContactInfoResponse(
                request.Name,
                request.Address,
                request.DateOfBirth,
                request.NationalInsuranceNumber);

            if (string.IsNullOrEmpty(request.Name))
            {
                response.AddError("Name cannot be empty");
            }

            if (string.IsNullOrEmpty(request.Address))
            {
                response.AddError("Address cannot be empty");
            }

            if (string.IsNullOrEmpty(request.NationalInsuranceNumber))
            {
                response.AddError("National Insurance number cannot be empty.");
            }

            var customerRecord = new Customer(request.Name, request.Address, request.DateOfBirth, request.NationalInsuranceNumber);

            if (customerRecord.Age < 18)
            {
                response.AddError("A customer must be over the age of 18");
            }

            if (response.HasError == false)
            {
                response.CreditScore = this._creditScoreService.GetCreditScore(request.NationalInsuranceNumber);

                if (response.CreditScore > 500)
                {
                    this._customerDatabase.Store(customerRecord);
                }
                else
                {
                    response.AddError("Credit score is too low, sorry!");
                }
            }

            return response;
        }
    }
}
```

I'm not going to talk through the code line by line, but you can see that the business use case is clear and explicit.

A new developer coming into the look at this class can see with reasonably little difficulty what is going on. It's clear and concise and very close to the actual business domain.

It's also not a huge stretch to imagine sitting down with a non-programmer and being able to talk through this code file line by line to iron out any issues.

So now we have the core of our business mapped (it's a very small bank), but how do we actually use the code.

## UI

Now we have the business logic worked out, we can start to think about how people at the bank can actually interact with the software.

In this case, the bank are more than happy having a small executable they can run, key in the information manually and wait for a result to come back.

Because all of our business logic is contained in our core library, the console app becomes really simple.

```csharp
// ------------------------------------------------------------
// Copyright (c) James Eastham.
// ------------------------------------------------------------

using System;
using CleanArchitecture.Core.Requests;

namespace CleanArchitecture.ConsoleApp
{
    /// <summary>
    /// Main application entry point.
    /// </summary>
    public static class Program
    {
        /// <summary>
        /// Main application entry point.
        /// </summary>
        /// <param name="args">Arguments passed into the application at runtime.</param>
        public static void Main(string[] args)
        {
            if (args == null)
            {
                throw new ArgumentException("args cannot be null");
            }

            var getContactInteractor = new GatherContactInfoInteractor(new MockCreditScoreService(), new CustomerDatabaseInMemoryImpl());

            Console.WriteLine("Name?");
            var name = Console.ReadLine();

            Console.WriteLine("Address?");
            var address = Console.ReadLine();

            Console.WriteLine("Date of birth (yyyy-MM-dd)?");
            var dateOfBirth = DateTime.Parse(Console.ReadLine());

            Console.WriteLine("NI Number?");
            var niNumber = Console.ReadLine();

            var getContactResponse = getContactInteractor.Handle(new GatherContactInfoRequest(name, address, dateOfBirth, niNumber));

            if (getContactResponse.HasError)
            {
                foreach (var error in getContactResponse.Errors)
                {
                    Console.WriteLine(error);
                }
            }
            else
            {
                var result = getContactResponse.IsAcceptedForLoan ? "accepted" : "rejected";

                Console.WriteLine($"Credit score is {getContactResponse.CreditScore} so customer has been {result}");
            }

            Console.WriteLine("Press any key to close");
            Console.ReadKey();
        }
    }
}
```

The UI is only concerned with how the data is presented to the user, it has no concern over how a customer gets validated.

If in 6 months the bank decides they want to allow potential customers to validate themselves through a website instead of manual entry the process is nice and simple.

The EXACT same library can be used, how that interface is served up is completely irrelevant.

Suddenly, this bank can be a lot more agile to changes outside of their control.

If a new technological trend kicks off and banks start identifying potential customers behind their backs (banks haven't heard of GDPR you see) they can again use the EXACT same library to run the validation.

It's also at runtime that the implementations of the CreditScoreService and the CustomersDatabase are injected.

# Summary

I've seen a lot of posts and comments around Clean Architecture stating it can be overkill for a lot of projects.

I do agree with this statement for small utility applications or any 'one-off' programs that will be used for a hotfix of any kind. However, I believe any applications that will be used in any kind of production scenario should be built in the right way from the ground up.

Small choices, like decoupling the database from the business logic, make the application so much more resilient to change.

