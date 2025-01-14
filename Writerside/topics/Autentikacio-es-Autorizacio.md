# Autentikáció és Autorizáció

[kép: https://docs.google.com/drawings/d/1bJSLzSSpIYbzLFG3vfwcmRSNhXIdJg1tVHnGF88dNWo/edit]

[bevezető]

## Authenticated API Command
Minden API Commandot a SharedModels-ben kell regisztrálni, ezzel biztosítva az elérhetőségét a kliens és a szerver projektből.

A autorizálni kívánt API commandnak implementálnia kell a ```AuthenticatedApiCommandBase<TResponse>```-t.
Ezzel biztosítva, hogy a megfelelő pipeline-ba kerül bele a parancs futtatásakor.

### GetRequiredPermissions()
A parancs végrehajtásához a user-től elvárt jogosultságokat tartalmazza.

Példa: 

```c#
public record AssignUserPermissionApiCommand : AuthenticatedApiCommandBase<AssignUserPermissionApiCommandResponse>
{
    public int Id { get; init; }
    public string PermissionSet { get; set; }

    public EntityPermissions GetPermissions()
    {
        return new EntityPermissions(PermissionSet);
    }

    public override string GetApiEndpoint()
    {
        return Path.Join(ApiBaseUrl, "/api/v1/iam/assign-permission");
    }

    public override HttpMethodEnum GetApiMethod()
    {
        return HttpMethodEnum.Put;
    }

    public override PermissionFlags[] GetRequiredPermissions()
    {
        return [PermissionFlags.CAN_ASSIGN_PERMISSIONS];
    }
}
```

# Input Validation
A commandhoz tartozó szabályokat tartalmazza. 
A szabályokat az API commandal megegyező fájlban kell regisztrálni.

```c#
public class AssignUserPermissionApiCommandValidator : AbstractValidator<AssignUserPermissionApiCommand>
{
    public AssignUserPermissionApiCommandValidator()
    {
        RuleFor(x => x.Id).NotEmpty();
    }
}
```

# JWT Validation
[kép: https://docs.google.com/drawings/d/13XkQAiiwz9wBqBG0p9inySpUsFH0HwIbohKoreI985A/edit]

