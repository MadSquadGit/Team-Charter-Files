## Setup

#### Base Interface
- Defines With methods and main method for execution
- Every with method
- Example
```c#
public interface IWorkflowChangeFluentService
{
	IWorkflowChangeFluentService WithId(int id);
	IWorkflowChangeFluentService WithTemplateNode(TemplateNode node);
	IWorkflowChangeFluentService WithLoggedInUser(string loggedInUser);
	IWorkflowChangeFluentService WithTargetStatus(string targetStatus = null);
	IWorkflowChangeFluentService WithApplicableUserRoles(List<UserRoleEnum> userRoles, bool isAdmin);
	IWorkflowChangeFluentService WithEmailRecipients(List<UserRoleEnum> emailRecipientRoles);
	TemplateNode ExecuteWorkflow();
}
```


#### Type Interface
- This creates a specific iteration of the Base Builder
- Example
```c#
public interface IReturnFluentService : IWorkflowChangeFluentService {}
```


#### Implemented Class
- This is the actial class that inplements the specific type interface
- This ios where teh logic for teh With methods and teh execute method/s will be places that is specific to this flow
- Example
```c#
public class ReturnFluentService : IReturnFluentService
{
	private readonly IRepository _Repository;
	private readonly IService _Service;
	private readonly IWorkflowUserRoleService _userRoleService;
	private readonly IEmailNotificationService _emailNotificationService;

	private int _Id;
	private string _loggedInUser;
	private string _targetStatus;
	private bool _isAdmin;
	private TemplateNode _templateNode;
	private List<UserRoleEnum> _userRoles;
	private List<UserRoleEnum> _emailRecipients;

	public ReturnFluentService(
		IRepository Repository,
		IService Service,
		IWorkflowUserRoleService userRoleService,
		IEmailNotificationService emailNotificationService)
	{
		_Repository = Repository;
		_Service = Service;
		_userRoleService = userRoleService;
		_emailNotificationService = emailNotificationService;
	}

	public IWorkflowChangeFluentService WithId(int id)
	{
		_Id = id;
		return this;
	}

	public IWorkflowChangeFluentService WithLoggedInUser(string loggedInUser)
	{
		_loggedInUser = loggedInUser;
		return this;
	}

	public IWorkflowChangeFluentService WithTargetStatus(string targetStatus = null)
	{
		_targetStatus = targetStatus;
		return this;
	}

	public IWorkflowChangeFluentService WithTemplateNode(TemplateNode node)
	{
		_templateNode = node;
		return this;
	}

	public IWorkflowChangeFluentService WithApplicableUserRoles(List<UserRoleEnum> userRoles, bool isAdmin)
	{
		_userRoles = userRoles;
		_isAdmin = isAdmin;
		return this;
	}

	public IWorkflowChangeFluentService WithEmailRecipients(List<UserRoleEnum> emailRecipientRoles)
	{
		_emailRecipients = emailRecipientRoles;
		return this;
	}


	public TemplateNode ExecuteWorkflow()
	{
		 original = _Service.GetById(_Id);

		if (original == null)
		{
			throw new KeyNotFoundException($" with Id {_Id} does not exist.");
		}

		if (!_userRoleService.UserHasRoles(original, _loggedInUser, _userRoles) && !_isAdmin)
		{
			throw new InvalidOperationException($"{_loggedInUser} does not have the valid role to perform this action");
		}

		if (_targetStatus == original.StatusCode)
		{
			throw new InvalidOperationException($"The  is already set to the requested status");
		}

		if (_targetStatus == Status.ReturnedByEnsTeam)
		{
			_Service.ResetAnsweredNode(_templateNode);
		}

		_Repository.Update(
			_Id,
			original.StatusCode,
			_targetStatus,
			_loggedInUser);


		 returned = _Service.GetById(_Id, true, true);

		// Send Emails
		List<string> recipients = _emailNotificationService.GetEmailRecipients(returned, _emailRecipients);

		_emailNotificationService.SendEmailAsync(returned, recipients)
			.GetAwaiter()
			.GetResult();

		return _Service.GetTemplateById(returned);
	}
}
```


#### Transient
- Add this for teh dependancy injection
- Needed to work and associate with the intefaces
- Add in `Startup.cs` File Services Transients
- Example
```cs
private static void SetupServices(IServiceCollection services)
{
 services.AddTransient<IReturnFluentService, ReturnFluentService>();
}
```

## Implementation
- Where the method is called

- Interface
```c#
public interface IWorkflowService
{
TemplateNode ReturnToCreditManager(int id, TemplateNode node, bool isAdmin);
}
```

- Class
```c#
public class WorkflowService : IWorkflowService
{
	private readonly IReturnFluentService _returnFluentService;

	public WorkflowService(
		IReturnFluentService returnFluentService)
	{
		_returnFluentService = returnFluentService;
	}

	public TemplateNode ReturnToCreditManager(int id, TemplateNode node, bool isAdmin)
	{
		return _returnFluentService
				.WithId(id)
				.WithTargetStatus(Status.ReturnedByEnsTeam)
				.WithLoggedInUser(_loggedInUser)
				.WithApplicableUserRoles(new List<UserRoleEnum> { UserRoleEnum.EnSTeam }, isAdmin)
				.WithTemplateNode(node)
				.WithEmailRecipients(new List<UserRoleEnum> { UserRoleEnum.CountryRepresentative, UserRoleEnum.EnSTeam, UserRoleEnum.Creator })
				.ExecuteWorkflow();
	}
}
```

