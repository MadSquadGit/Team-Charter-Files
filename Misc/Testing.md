
## Prelude
- Test where the complexity is
- Testing comes in at least 3's
	- Happy Path
	- Sad Path
	- Unexpected Path
- Create at least 3 tests foe these paths
- Can create more

## AutoFixture and AutoMoq

```c#
 public async Task ESReport_Given_Identifier_When_GetLatestEsReportForCustomer_Then_Return_ESReport(
            IFixture fixture,
            [Frozen] Mock<IEsReportService> esReportServiceMock,
            [Frozen] Mock<IAssessmentService> assessmentServiceMock)
        {
            fixture.Customize(new AutoMoqCustomization());

            var staticServiceMock = new Mock<IStaticService>();
            var assessmentSummaryServiceMock = new Mock<IAssessmentSummaryService>();

            fixture.Inject(staticServiceMock.Object);
            fixture.Inject(assessmentSummaryServiceMock.Object);

            staticServiceMock.Setup(x => x.GetSectionInputTypes()).Returns(FakeSectionInput.GetInputTypesFake());
            esReportServiceMock.Setup(x => x.GetReportByAssessmentId(It.IsAny<int>())).ReturnsAsync(FakeEsReport.GetEsReportFake());
            assessmentSummaryServiceMock.Setup(x => x.ListByAssessmentId(It.IsAny<int>())).ReturnsAsync(FakeAssessmentSummary.GetAssessmentSummariesFake());
            assessmentServiceMock.Setup(x => x.GetLatestAssessmentByCustomerIdentifier(It.IsAny<string>())).Returns(FakeAssessment.GetExistingCompletedFakeAssessment());

            var resultsService = fixture.Create<ResultsService>();

            var response = await resultsService.GetLatestEsReportForCustomer("001TestCustomerCode");

            Assert.NotNull(response.Data);
            Assert.IsType<EsReport>(response.Data);
            Assert.Equal("Content", response.Data.Conclusion);
        }
```

