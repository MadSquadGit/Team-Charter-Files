
## Prelude
- Test where the complexity is
- Testing comes in at least 3's
	- Happy Path
	- Sad Path
	- Unexpected Path
- Create at least 3 tests foe these paths
- Can create more

## AutoFixture and AutoMoq

```csharp
 public async Task given_when(
            IFixture fixture,
            [Frozen] Mock<IService> eserviceMock,
            [Frozen] Mock<IAService> aServiceMock)
        {
            fixture.Customize(new AutoMoqCustomization());

            var staticServiceMock = new Mock<IStaticService>();
            var aSummaryServiceMock = new Mock<IASummaryService>();

            fixture.Inject(staticServiceMock.Object);
            fixture.Inject(aSummaryServiceMock.Object);

            staticServiceMock.Setup(x => x.GetInputTypes()).Returns(FakeSectionInput.GetInputTypesFake());
            serviceMock.Setup(x => x.GetById(It.IsAny<int>())).ReturnsAsync(FakeReport.GetReportFake());
            aSummaryServiceMock.Setup(x => x.ListById(It.IsAny<int>())).ReturnsAsync(FakeASummary.GetSummariesFake());
            aServiceMock.Setup(x => x.GetLatestByIdentifier(It.IsAny<string>())).Returns(FakeAssessment.GetExistingCompletedFake());

            var resultsService = fixture.Create<ResultsService>();

            var response = await resultsService.GetLatestReportForCustomer("001TestCustomerCode");

            Assert.NotNull(response.Data);
            Assert.IsType<Report>(response.Data);
            Assert.Equal("Content", response.Data.Conclusion);
        }
```

