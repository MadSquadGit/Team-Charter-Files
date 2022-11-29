
```toc
```


## Builder

### Setup

#### Base Interface
- Defines With methods and main method for execution
- Every with method
- Example
```c#
public interface IWorkflowChangeFluentService
{
	IWorkflowChangeFluentService WithAssessmentId(int id);
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
public interface IReturnAssessmentFluentService : IWorkflowChangeFluentService {}
```


#### Implemented Class
- This is the actial class that inplements the specific type interface
- This ios where teh logic for teh With methods and teh execute method/s will be places that is specific to this flow
- Example
```c#
public class ReturnAssessmentFluentService : IReturnAssessmentFluentService
{
	private readonly IAssessmentRepository _assessmentRepository;
	private readonly IAssessmentService _assessmentService;
	private readonly IWorkflowUserRoleService _userRoleService;
	private readonly IEmailNotificationService _emailNotificationService;

	private int _assessmentId;
	private string _loggedInUser;
	private string _targetStatus;
	private bool _isAdmin;
	private TemplateNode _templateNode;
	private List<UserRoleEnum> _userRoles;
	private List<UserRoleEnum> _emailRecipients;

	public ReturnAssessmentFluentService(
		IAssessmentRepository assessmentRepository,
		IAssessmentService assessmentService,
		IWorkflowUserRoleService userRoleService,
		IEmailNotificationService emailNotificationService)
	{
		_assessmentRepository = assessmentRepository;
		_assessmentService = assessmentService;
		_userRoleService = userRoleService;
		_emailNotificationService = emailNotificationService;
	}

	public IWorkflowChangeFluentService WithAssessmentId(int id)
	{
		_assessmentId = id;
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
		Assessment originalAssessment = _assessmentService.GetAssessmentById(_assessmentId);

		if (originalAssessment == null)
		{
			throw new KeyNotFoundException($"Assessment with Id {_assessmentId} does not exist.");
		}

		if (!_userRoleService.UserHasRoles(originalAssessment, _loggedInUser, _userRoles) && !_isAdmin)
		{
			throw new InvalidOperationException($"{_loggedInUser} does not have the valid role to perform this action");
		}

		if (_targetStatus == originalAssessment.StatusCode)
		{
			throw new InvalidOperationException($"The assessment is already set to the requested status");
		}

		if (_targetStatus == Status.ReturnedByEnsTeam)
		{
			_assessmentService.ResetAnsweredNode(_templateNode);
		}

		_assessmentRepository.UpdateAssessment(
			_assessmentId,
			originalAssessment.StatusCode,
			_targetStatus,
			_loggedInUser);


		Assessment returnedAssessment = _assessmentService.GetAssessmentById(_assessmentId, true, true);

		// Send Emails
		List<string> recipients = _emailNotificationService.GetEmailRecipients(returnedAssessment, _emailRecipients);

		_emailNotificationService.SendAssessmentEmailAsync(returnedAssessment, recipients)
			.GetAwaiter()
			.GetResult();

		return _assessmentService.GetAssessmentTemplateById(returnedAssessment);
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
 services.AddTransient<IReturnAssessmentFluentService, ReturnAssessmentFluentService>();
}
```

### Implementation
- Where the method is called

- Interface
```c#
public interface IAssessmentWorkflowService
{
TemplateNode ReturnAssessmentToCreditManager(int id, TemplateNode node, bool isAdmin);
}
```

- Class
```c#
public class AssessmentWorkflowService : IAssessmentWorkflowService
{
	private readonly IReturnAssessmentFluentService _returnAssessmentFluentService;

	public AssessmentWorkflowService(
		IReturnAssessmentFluentService returnAssessmentFluentService)
	{
		_returnAssessmentFluentService = returnAssessmentFluentService;
	}

	public TemplateNode ReturnAssessmentToCreditManager(int id, TemplateNode node, bool isAdmin)
	{
		return _returnAssessmentFluentService
				.WithAssessmentId(id)
				.WithTargetStatus(Status.ReturnedByEnsTeam)
				.WithLoggedInUser(_loggedInUser)
				.WithApplicableUserRoles(new List<UserRoleEnum> { UserRoleEnum.EnSTeam }, isAdmin)
				.WithTemplateNode(node)
				.WithEmailRecipients(new List<UserRoleEnum> { UserRoleEnum.CountryRepresentative, UserRoleEnum.EnSTeam, UserRoleEnum.AssessmentCreator })
				.ExecuteWorkflow();
	}
}
```

