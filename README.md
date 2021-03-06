# On-boarding Task
--
Task 1.a Display a list of all property names and their property id’s for Owner Id: 1426

    SELECT p.[Name] AS PropertyName,   
           p.Id AS PropertyID  
    FROM OwnerProperty AS op  
    INNER JOIN Property AS p ON op.PropertyId=p.Id  
    WHERE op.OwnerId=1426;  
![1](https://user-images.githubusercontent.com/82072152/160957177-a80885bb-a4d8-49a5-9ac5-810122a0dd52.png)

--
Task 1.b Display the current home value for each property in question a)

    SELECT p.[Name] AS PropertyName,  
           p.Id AS PropertyID,  
           phv.Value AS CurrentHomeValue  
    FROM OwnerProperty AS op  
    INNER JOIN Property AS p ON op.PropertyId=p.Id  
    INNER JOIN PropertyHomeValue AS phv	ON p.Id=phv.PropertyId  
    WHERE op.OwnerId=1426 AND phv.IsActive=1  
    ORDER BY p.[Name];  
![2](https://user-images.githubusercontent.com/82072152/160957179-0c0d14e5-a72a-495d-9785-f04caf05bb61.png)
 
 --
Task 1.c For each property in question a), return the following:  
i) Using rental payment amount, rental payment frequency, tenant start date and tenant end date to write a query that returns the sum of all payments from start date to end date.

    SELECT p.id, p.Name, tp.PaymentAmount, tp.StartDate, tp.EndDate, tp.PaymentFrequencyId, trt.Name, 
    CASE WHEN trt.Name = 'Weekly' THEN tp.PaymentAmount*52
         WHEN trt.Name = 'Fortnightly' THEN tp.PaymentAmount*26
         WHEN trt.Name = 'Monthly' THEN tp.PaymentAmount*12
    END/52 * DATEDIFF(Week,tp.StartDate,tp.EndDate) as TotalPayment
    FROM [dbo].[Property] p 
    Inner join [dbo].[OwnerProperty] op on p.id = op.PropertyId
    Inner join [dbo].[TenantProperty] tp on p.id = tp.PropertyId
    Inner join [dbo].[TenantPaymentFrequencies] tpf on tpf.Id = tp.PaymentFrequencyId
    Inner join [dbo].[TargetRentType] trt on trt.Id = tp.PaymentFrequencyId
    WHERE op.OwnerId = 1426
![3](https://user-images.githubusercontent.com/82072152/160957182-ef447f14-881e-4dcd-8add-c2d91c33c060.png)

--
Task 1.c For each property in question a), return the following:  
ii) Display the yield.


    SELECT p.[Name] AS PropertyName,  
           p.Id AS PropertyID,  
           phv.[Value] AS HomeValue,  
           (  
               (  
                   (CASE WHEN trt.[Name] ='Weekly' THEN (DATEDIFF(wk,tp.StartDate, tp.EndDate)*prp.Amount)  
                         WHEN trt.[Name] ='Fortnightly' THEN ((DATEDIFF(wk,tp.StartDate, tp.EndDate)/2)*prp.Amount)  
                         ELSE (DATEDIFF(m,tp.StartDate, tp.EndDate)*prp.Amount)  
                    END)  
               -ISNULL(SUM(pe.Amount),0)  
               )/phv.[Value]  
            )*100 AS Yield,  
            trt.[Name] AS RentalPaymentFrequency,  
            prp.Amount AS RentalPaymentAmount,  
            tp.StartDate AS TenantStartDate,  
            tp.EndDate AS TenantEndDate 
    FROM Property AS p  
    INNER JOIN OwnerProperty AS op ON p.Id=op.PropertyId  
    INNER JOIN PropertyRentalPayment AS prp ON p.Id=prp.PropertyId  
    INNER JOIN TargetRentType AS trt ON prp.FrequencyType=trt.Id  
    INNER JOIN TenantProperty AS tp ON p.Id=tp.PropertyId  
    INNER JOIN PropertyHomeValue AS phv	ON p.Id=phv.PropertyId  
    LEFT JOIN PropertyExpense AS pe ON p.Id=pe.PropertyId  
    WHERE op.OwnerId=1426 AND phv.IsActive=1  
    GROUP BY p.[Name],p.Id,phv.[Value],trt.[Name],prp.Amount,tp.StartDate,tp.EndDate;  
![4](https://user-images.githubusercontent.com/82072152/160957186-19e8db40-c0f3-422c-a8af-14d63e809393.png)

--
Task 1.d.	Display all the jobs available in the marketplace (jobs that owners have advertised for service suppliers).

    SELECT DISTINCT j.Id AS JobID,  
           j.PropertyId,  
           j.OwnerId,  
           j.JobDescription,  
           jm.IsActive 
    FROM Job AS j  
    INNER JOIN JobMedia AS jm ON j.Id=jm.JobId  
    WHERE jm.IsActive=1  
![5](https://user-images.githubusercontent.com/82072152/160957188-760d1e17-d864-449e-8237-723e5c3f3f1d.png)

--
Task 1.e.	Display all property names, current tenants first and last names and rental payments per week/ fortnight/month for the properties in question a).

    SELECT p.[Name] AS PropertyName,  
           Person.FirstName AS TenantFirstName,  
           Person.LastName AS TenantLastName,  
           prp.Amount AS RentalPaymentAmount,  
           trt.[Name] AS RentalPaymentFrequency 
    FROM OwnerProperty AS op  
    INNER JOIN Property AS p ON op.PropertyId=p.Id  
    INNER JOIN TenantProperty AS tp ON p.Id=tp.PropertyId  
    INNER JOIN Tenant AS t ON tp.TenantId=t.Id  
    INNER JOIN Person ON t.Id=Person.Id  
    INNER JOIN PropertyRentalPayment AS prp ON p.Id=prp.PropertyId  
    INNER JOIN TargetRentType AS trt ON prp.FrequencyType=trt.Id  
    WHERE op.OwnerId=1426;  
![6](https://user-images.githubusercontent.com/82072152/160957189-a457582d-e24c-412c-911e-6c228157eeae.png)

--Task 2 Use Report Builder or Visual Studio (SSRS) to develop the following report:
![7](https://user-images.githubusercontent.com/82072152/160957191-3b0cf858-e41e-4d78-bccb-3fdc08c7ece9.png)

 Solution, Data Set Query: 
 
    Select p.Id,p.[Name] AS PropertyName,  
           RTRIM(LTRIM(  
                        CONCAT(  
                               COALESCE(Person.FirstName + ' ', ''),  
                               COALESCE(Person.MiddleName + ' ', ''),  
                               COALESCE(Person.Lastname, '')  
                              )  
                       )  
                 ) AS OwnerName,   
            RTRIM(LTRIM(  
                        CONCAT(  
                               COALESCE(a.Number + ' ', ''),  
                               COALESCE(a.Street + ' ', ''),  
                               COALESCE(a.Suburb + ' ', ''),  
                               COALESCE(a.PostCode + ' ', ''),  
                               COALESCE(a.Region + ' ', ''),  
                               COALESCE(Country.[Name], '')  
                              )  
                        )  
                   ) AS PropertyAddress,   
            CONCAT(p.Bathroom,' Bathroom, ',p.Bedroom,' Bedroom, ',p.ParkingSpace,' ParkingSpace') AS PropertyDetails,  
            prp.Amount as RentalAmount,  
            CASE WHEN trt.[Name] ='Weekly' THEN 'Week'  
                 WHEN trt.[Name] ='Fortnightly' THEN 'Fortnight' ELSE 'Month'  
            END AS RentalPaymentFrequency,  
            e.[Description],  
            e.Amount,  
            e.[Date]  
    FROM Property AS p  
    INNER JOIN OwnerProperty AS op ON p.Id=op.PropertyId  
    INNER JOIN Owners AS o ON op.OwnerId=o.Id  
    INNER JOIN Person ON o.Id=Person.Id  
    INNER JOIN [Address] AS a ON p.AddressId=a.AddressId  
    INNER JOIN Country ON a.CountryId=Country.Id  
    INNER JOIN PropertyRentalPayment AS prp ON p.Id=prp.PropertyId  
    INNER JOIN TargetRentType as trt ON prp.FrequencyType=trt.Id  
    LEFT JOIN PropertyExpense AS e ON p.Id=e.PropertyId  
    WHERE p.[Name]=@Property  
    
![8](https://user-images.githubusercontent.com/82072152/161162328-f5068627-20a3-4af8-9c25-a418e36a63aa.png)

![9](https://user-images.githubusercontent.com/82072152/161162285-b81b2ef8-31bc-448d-af13-afa2081d300a.png)
