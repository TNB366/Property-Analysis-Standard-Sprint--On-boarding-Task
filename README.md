-- Task 1.a Display a list of all property names and their property id’s for Owner Id: 1426
SELECT p.id, p.Name from [dbo].[Property] p, [dbo].[OwnerProperty] op
WHERE p.Id = op.PropertyId and op.OwnerId = 1426
