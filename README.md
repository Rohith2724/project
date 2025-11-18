        {
            // Filename: MVRngControllerTests.cs
using System.Reflection;
using System.Security.Claims;
using com.gaig.mc.MVR.Controllers;
using com.gaig.mc.MVR.Records;
using com.gaig.mc.MVR.Services;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging.Abstractions;
using Moq;
using NUnit.Framework;
using static com.gaig.mc.MVR.Utilities.UserDtoSecurityExtensions;

namespace com.gaig.mc.MVR.test.Controllers
{
    [TestFixture]
    public class MVRngControllerTests
    {
        private MVRngController TestSubject { get; set; } = null!;
        private Mock<IMVRngService> MockMvrService { get; set; } = null!;
        private Mock<IMaintCenterService> MockMaintCenterService { get; set; } = null!;

        [SetUp]
        public void SetUp()
        {
            MockMvrService = new Mock<IMVRngService>();
            MockMaintCenterService = new Mock<IMaintCenterService>();
            var assemblyName = new AssemblyName("MVRWebApi") { Version = new Version("3.0.0.1") };

            TestSubject = new MVRngController(
                new NullLogger<MVRngController>(),
                MockMvrService.Object,
                MockMaintCenterService.Object,
                assemblyName
            );

            SetupUserContext(); // default context for tests
        }

        private void SetupUserContext(string? vid = "testUser", string? role = "EDIT_ROLE")
        {
            var claims = new List<Claim>
            {
                new Claim("user_attributes", $"{{\"VID\": [\"{vid}\"] }}"),
                new Claim("name", "Test Name"),
                new Claim("email", "test@gaig.com"),
                new Claim("given_name", "Test"),
                new Claim(ClaimTypes.Role, role ?? "EDIT_ROLE")
            };
            var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
            var principal = new ClaimsPrincipal(identity);

            var ctx = new DefaultHttpContext { User = principal };
            TestSubject.ControllerContext = new ControllerContext { HttpContext = ctx };
        }

        // -------------------------
        // Previously covered tests...
        // (keep your earlier tests here as needed)
        // -------------------------

        [Test]
        public void PostBulkOrder_CallsServiceAndReturnsOk()
        {
            // Arrange
            var id = "bulk1";
            var newOrder = new NewOrderDTO
            {
                businessUnit = "BU1",
                profitCenter = "PC1",
                trackingId = "T1",
                drivers = new List<NewDriverDTO>
                {
                    new() { firstName = "A", lastName = "B", stateCode = "NY", dateOfBirth = DateTime.UtcNow, licenseNumber = "L1", requestId = "R1" }
                }
            };
            var expected = new Result { errorCode = 0, errorMessage = "" };
            // Service method signature: PostBulkOrder(string id, NewOrderDTO bulkOrderDTO, UserDto userDto)
            MockMvrService.Setup(s => s.PostBulkOrder(id, It.IsAny<NewOrderDTO>(), It.IsAny<UserDto>())).Returns(expected);

            // Act
            var res = TestSubject.PostBulkOrder(id, newOrder);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj, Is.Not.Null);
            Assert.That(obj!.Value, Is.EqualTo(expected));
            MockMvrService.Verify(s => s.PostBulkOrder(id, It.IsAny<NewOrderDTO>(), It.IsAny<UserDto>()), Times.Once);
        }

        [Test]
        public void GetAllFilteredOrders_CallsServiceAndReturnsOk()
        {
            // Arrange
            var id = "o1";
            var filter = new NewOrderFilterDTO
            {
                requestedBy = "U1",
                businessUnit = "BU1",
                startDate = DateTime.UtcNow.AddDays(-7),
                endDate = DateTime.UtcNow,
                drivers = new List<Driver> { new() { firstName = "D1", lastName = "L1", stateCode = "NY" } }
            };
            var expectedList = new List<MotorVehicleRecord> { new() };
            MockMvrService.Setup(s => s.GetAllFilteredOrders(id, filter)).Returns(expectedList);

            // Act
            var res = TestSubject.GetAllFilteredOrders(id, filter);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expectedList));
            MockMvrService.Verify(s => s.GetAllFilteredOrders(id, filter), Times.Once);
        }

        [Test]
        public void GetAllInvoiceOrders_CallsServiceAndReturnsOk()
        {
            // Arrange
            var id = "inv1";
            var filter = new NewOrderFilterDTO
            {
                requestedBy = "U2",
                businessUnit = "BU2",
                drivers = new List<Driver>()
            };
            var expectedList = new List<MotorVehicleRecord> { new() };
            MockMvrService.Setup(s => s.GetAllInvoiceOrders(id, filter)).Returns(expectedList);

            // Act
            var res = TestSubject.GetAllInvoiceOrders(id, filter);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expectedList));
            MockMvrService.Verify(s => s.GetAllInvoiceOrders(id, filter), Times.Once);
        }

        [Test]
        public void GetAllFilteredDrivers_CallsServiceAndReturnsOk()
        {
            // Arrange
            var id = "drv1";
            var driverFilter = new NewDriverFilter
            {
                requestedBy = "U3",
                businessUnit = "BU3",
                includeInactives = false,
                drivers = new List<Driver> { new() { firstName = "F", lastName = "L", stateCode = "CA" } }
            };
            var expected = new List<MotorVehicleRecord> { new() };
            MockMvrService.Setup(s => s.GetAllFilteredDrivers(id, driverFilter)).Returns(expected);

            // Act
            var res = TestSubject.GetAllFilteredDrivers(id, driverFilter);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expected));
            MockMvrService.Verify(s => s.GetAllFilteredDrivers(id, driverFilter), Times.Once);
        }

        [Test]
        public void UpdateDriverStatus_CallsServiceAndReturnsOk()
        {
            // Arrange
            var id = "upd1";
            var dto = new DriverUpdateRecord { requestId = "R1", status = "OK" };
            var expected = new Result { errorCode = 0 };
            MockMvrService.Setup(s => s.UpdateDriverStatus(id, dto)).Returns(expected);

            // Act
            var res = TestSubject.UpdateDriverStatus(id, dto);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expected));
            MockMvrService.Verify(s => s.UpdateDriverStatus(id, dto), Times.Once);
        }

        [Test]
        public void PushPdfToMyFile_CallsServiceAndReturnsOk()
        {
            // Arrange
            var id = "pf1";
            var myFile = new MyFileExport { mvrId = "1", pdf = new byte[] { 1, 2 } };
            var expected = new Result { errorCode = 0 };
            MockMvrService.Setup(s => s.PushPdfToMyFile(id, myFile)).Returns(expected);

            // Act
            var res = TestSubject.PushPdfToMyFile(id, myFile);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expected));
            MockMvrService.Verify(s => s.PushPdfToMyFile(id, myFile), Times.Once);
        }

        // You can add more tests here for exceptional paths for above endpoints (throwing in service),
        // if you want to exercise controller catch blocks for each method.
    }
}
 = new Mock<IMVRngService>();
            MockMaintCenterService = new Mock<IMaintCenterService>();
            AssemblyName assemblyName = new("MVRWebApi") { Version = new Version("3.0.0.1") };

            TestSubject = new MVRngController(
                new NullLogger<MVRngController>(),
                MockMvrService.Object,
                MockMaintCenterService.Object,
                assemblyName
            );

            SetupUserContext(); // default user context used by many tests
        }

        private void SetupUserContext(string? role = null, string? vid = "testUser")
        {
            // Build claims that the UserDto extension reads
            var claimsList = new List<Claim>
            {
                new Claim("user_attributes", $"{{\"VID\": [\"{vid}\"] }}"),
                new Claim("name", "Test Name"),
                new Claim("email", "test@gaig.com"),
                new Claim("given_name", "Test"),
                new Claim(ClaimTypes.Role, role ?? "EDIT_ROLE")
            };

            var identity = new ClaimsIdentity(claimsList, CookieAuthenticationDefaults.AuthenticationScheme);
            var principal = new ClaimsPrincipal(identity);

            var ctx = new DefaultHttpContext { User = principal };
            TestSubject.ControllerContext = new ControllerContext { HttpContext = ctx };
        }

        [Test]
        public void GetAuthorizationDTO_ReturnsAuthorizationData_FromServiceWithUserDto()
        {
            // Arrange - Setup service to expect any UserDto and return values
            MockMvrService.Setup(s => s.hasAdminAccess(It.IsAny<UserDto>())).Returns(true);
            MockMvrService.Setup(s => s.IsAgent(It.IsAny<UserDto>())).Returns(true);
            MockMvrService.Setup(s => s.hasAltMarketAccess(It.IsAny<UserDto>())).Returns(false);
            MockMvrService.Setup(s => s.hasOceanMarineAccess(It.IsAny<UserDto>())).Returns(true);
            MockMvrService.Setup(s => s.hasOutsideSupportAccess(It.IsAny<UserDto>())).Returns(false);
            MockMvrService.Setup(s => s.hasRequestAccess(It.IsAny<UserDto>())).Returns(true);
            MockMvrService.Setup(s => s.hasLicenseAdminAccess(It.IsAny<UserDto>())).Returns(true);
            MockMvrService.Setup(s => s.hasOMProfitCenterAdminAccess(It.IsAny<UserDto>())).Returns(false);

            // Act
            var actionResult = TestSubject.GetAuthorizationDTO();

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj, Is.Not.Null);
            Assert.That(obj!.Value, Is.InstanceOf<NewAuthorizationDTO>());

            var dto = obj.Value as NewAuthorizationDTO;
            Assert.That(dto!.IsAdmin, Is.True);
            Assert.That(dto.IsAgent, Is.True);
            Assert.That(dto.IsAltMarket, Is.False);
            Assert.That(dto.IsOceanMarine, Is.True);
            Assert.That(dto.IsOutsideSupport, Is.False);
            Assert.That(dto.IsRequest, Is.True);
            Assert.That(dto.IsLicenseAdmin, Is.True);
            Assert.That(dto.IsOMProfitCenterAdmin, Is.False);

            // Verify service calls happened with a UserDto (we can't directly inspect the UserDto instance, so verify invocation)
            MockMvrService.Verify(s => s.hasAdminAccess(It.IsAny<UserDto>()), Times.Once);
            MockMvrService.Verify(s => s.IsAgent(It.IsAny<UserDto>()), Times.Once);
        }

        [Test]
        public void GetListStates_ReturnsKeyValuePairs_FromServiceDictionary()
        {
            // Arrange
            var dict = new Dictionary<string, string> { { "NY", "New York" }, { "CA", "California" } };
            MockMvrService.Setup(s => s.GetStatesList()).Returns(dict);

            // Act
            var actionResult = TestSubject.GetListStates();

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            MockMvrService.Verify(s => s.GetStatesList(), Times.Once);
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value, Is.InstanceOf<IEnumerable<KeyValuePair<string, string>>>());
            var value = obj.Value as IEnumerable<KeyValuePair<string, string>>;
            Assert.That(value, Is.Not.Null);
            // convert and check presence
            var list = value!.ToList();
            Assert.That(list.Any(kv => kv.Key == "NY" && kv.Value == "New York"), Is.True);
        }

        [Test]
        public void GetAllStateValidationInfo_ReturnsFromService()
        {
            // Arrange
            var map = new Dictionary<string, StateInfo> { { "NY", new StateInfo { stateCode = "NY" } } };
            MockMvrService.Setup(s => s.GetAllStateInfo()).Returns(map);

            // Act
            var actionResult = TestSubject.GetAllStateValidationInfo();

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(map));
            MockMvrService.Verify(s => s.GetAllStateInfo(), Times.Once);
        }

        [Test]
        public void GetStateLicenseFormat_ReturnsFromService()
        {
            // Arrange
            var expected = new List<string> { "F1", "F2" };
            MockMvrService.Setup(s => s.GetStateLicenseFormat("NY")).Returns(expected);

            // Act
            var actionResult = TestSubject.GetStateLicenseFormat("NY");

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expected));
            MockMvrService.Verify(s => s.GetStateLicenseFormat("NY"), Times.Once);
        }

        [Test]
        public void GetListBusinessUnits_ReturnsFromService()
        {
            // Arrange
            var expected = new List<busUnitDTO> { new() { busUnitId = 1, busUnitName = "BU" } };
            MockMvrService.Setup(s => s.GetBusinessUnits()).Returns(expected);

            // Act
            var actionResult = TestSubject.GetListBusinessUnits();

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(expected));
            MockMvrService.Verify(s => s.GetBusinessUnits(), Times.Once);
        }

        [Test]
        public void GetListProfitCenter_ReturnsFromService()
        {
            // Arrange
            var arr = new string[] { "PC1", "PC2" };
            MockMvrService.Setup(s => s.GetListProfitCenter()).Returns(arr);

            // Act
            var actionResult = TestSubject.GetListProfitCenter();

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(arr));
            MockMvrService.Verify(s => s.GetListProfitCenter(), Times.Once);
        }

        [Test]
        public void GetUserDTO_ReturnsUserDto_FromClaims()
        {
            // Arrange - SetupUserContext already adds claims that map into a UserDto
            // Act
            var actionResult = TestSubject.GetUserDTO();

            // Assert
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value, Is.InstanceOf<UserDto>());

            var user = obj.Value as UserDto;
            Assert.That(user!.UserId, Is.EqualTo("testUser"));
            Assert.That(user.Email, Is.EqualTo("test@gaig.com"));
            Assert.That(user.FullName, Is.EqualTo("Test Name"));
        }

        [Test]
        public void GetUserDTO_ReturnsFailureMessage_WhenUserNull_AndControllerCatches()
        {
            // Arrange: set ControllerContext user to null to force extension to throw an exception
            TestSubject.ControllerContext = new ControllerContext { HttpContext = new DefaultHttpContext() { User = null! } };

            // Act
            var actionResult = TestSubject.GetUserDTO();

            // Assert - controller catches error and returns Ok with failure message
            Assert.That(actionResult.StatusCode, Is.EqualTo(200));
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.Value.ToString(), Does.StartWith("failure:"));
        }

        [Test]
        public void GetUserInfo_ReturnsBadRequest_WhenUsernameEmpty()
        {
            // Act
            var actionResult = TestSubject.GetUserInfo("   ");

            // Assert
            Assert.That(actionResult, Is.TypeOf<BadRequestObjectResult>());
            var bad = actionResult as BadRequestObjectResult;
            Assert.That(bad!.Value, Is.EqualTo("Username is required"));
        }

        [Test]
        public void GetUserInfo_ReturnsNotFound_WhenServiceReturnsNull()
        {
            // Arrange
            var username = "unknown";
            MockMvrService.Setup(s => s.GetUserByUserName(username)).Returns((UserDto?)null);

            // Act
            var actionResult = TestSubject.GetUserInfo(username);

            // Assert
            Assert.That(actionResult, Is.TypeOf<NotFoundObjectResult>());
            var nf = actionResult as NotFoundObjectResult;
            Assert.That(nf!.Value, Is.EqualTo($"User '{username}' not found"));
            MockMvrService.Verify(s => s.GetUserByUserName(username), Times.Once);
        }

        [Test]
        public void GetUserInfo_ReturnsOk_WhenServiceReturnsUser()
        {
            // Arrange
            var username = "jdoe";
            var userDto = new UserDto { UserId = "jdoe", FullName = "John Doe" };
            MockMvrService.Setup(s => s.GetUserByUserName(username)).Returns(userDto);

            // Act
            var actionResult = TestSubject.GetUserInfo(username);

            // Assert
            Assert.That(actionResult, Is.TypeOf<OkObjectResult>());
            var ok = actionResult as OkObjectResult;
            Assert.That(ok!.Value, Is.EqualTo(userDto));
        }

        [Test]
        public void GetUserInfo_Returns500_OnExceptionFromService()
        {
            // Arrange
            var username = "err";
            MockMvrService.Setup(s => s.GetUserByUserName(username)).Throws(new Exception("boom"));

            // Act
            var actionResult = TestSubject.GetUserInfo(username);

            // Assert
            Assert.That(actionResult, Is.TypeOf<ObjectResult>());
            var obj = actionResult as ObjectResult;
            Assert.That(obj!.StatusCode, Is.EqualTo(500));
            Assert.That(obj.Value, Is.EqualTo("Exception error on UserInfo action in controller"));
        }

        [Test]
        public void GetBusinessUnit_ReturnsValue_AndHandlesException()
        {
            // Arrange - success
            MockMvrService.Setup(s => s.GetBusinessUnit()).Returns("BU-Name");

            var okResult = TestSubject.GetBusinessUnit();
            Assert.That(okResult.StatusCode, Is.EqualTo(200));
            var obj = okResult as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo("BU-Name"));

            // Arrange - exception path
            MockMvrService.Setup(s => s.GetBusinessUnit()).Throws(new Exception("failBU"));

            var failResult = TestSubject.GetBusinessUnit();
            Assert.That(failResult.StatusCode, Is.EqualTo(200));
            var obj2 = failResult as ObjectResult;
            Assert.That(obj2!.Value, Is.EqualTo($"failure: failBU"));
        }

        [Test]
        public void PostOrder_PostsAndReturnsResult()
        {
            // Arrange
            var id = "id1";
            var orderDto = new NewOrderDTO { businessUnit = "BU" };
            var resultVal = new Result { errorCode = 0, errorMessage = "" };
            MockMvrService.Setup(s => s.PostOrder(id, orderDto, It.IsAny<UserDto>())).Returns(resultVal);

            // Act
            var res = TestSubject.PostOrder(id, orderDto);

            // Assert
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(resultVal));
            MockMvrService.Verify(s => s.PostOrder(id, orderDto, It.IsAny<UserDto>()), Times.Once);
        }

        [Test]
        public void PostOrderImport_ReturnsOk_OrFailureHandled()
        {
            var id = "id";
            var importOrder = new ImportOrder();
            var okRes = new Result { errorCode = 0 };

            MockMvrService.Setup(s => s.PostOrderImport(id, importOrder)).Returns(okRes);
            var r = TestSubject.PostOrderImport(id, importOrder);
            Assert.That(r.StatusCode, Is.EqualTo(200));
            var obj = r as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(okRes));

            // exception path
            MockMvrService.Setup(s => s.PostOrderImport(id, importOrder)).Throws(new Exception("impFail"));
            var r2 = TestSubject.PostOrderImport(id, importOrder);
            Assert.That(r2.StatusCode, Is.EqualTo(200));
            var obj2 = r2 as ObjectResult;
            Assert.That(obj2!.Value, Is.EqualTo($"failure: impFail"));
        }

        [Test]
        public void AdminEndpoints_GetAllLicenseRuleList_And_ErrorBranch()
        {
            var rules = new List<LicenseRuleDTO> { new() { stateCode = "NY", licenseRule = "R" } };
            MockMvrService.Setup(s => s.GetAllLicenseRuleList()).Returns(rules);

            var res = TestSubject.GetAllLicenseRuleList();
            Assert.That(res.StatusCode, Is.EqualTo(200));
            var obj = res as ObjectResult;
            Assert.That(obj!.Value, Is.EqualTo(rules));

            // Exception branch
            MockMvrService.Setup(s => s.GetAllLicenseRuleList()).Throws(new Exception("ruleErr"));
            var res2 = TestSubject.GetAllLicenseRuleList();
            Assert.That(res2.StatusCode, Is.EqualTo(200));
            var obj2 = res2 as ObjectResult;
            Assert.That(obj2!.Value, Is.EqualTo($"failure: ruleErr"));
        }

        [Test]
        public void AdminEndpoints_Tooltip_AddDeleteProfitCenter_SaveProfitCenter_ExceptionAndSuccessPaths()
        {
            // GetAllStateLicenseTooltipList success & failure
            var tooltipList = new List<StateInfo> { new() { stateCode = "NY" } };
            MockMvrService.Setup(s => s.GetAllStateLicenseTooltipList()).Returns(tooltipList);
            var tRes = TestSubject.GetAllStateLicenseTooltipList();
            Assert.That(tRes.StatusCode, Is.EqualTo(200));
            var tObj = tRes as ObjectResult;
            Assert.That(tObj!.Value, Is.EqualTo(tooltipList));

            MockMvrService.Setup(s => s.GetAllStateLicenseTooltipList()).Throws(new Exception("tipErr"));
            var tRes2 = TestSubject.GetAllStateLicenseTooltipList();
            Assert.That(tRes2.StatusCode, Is.EqualTo(200));
            var tObj2 = tRes2 as ObjectResult;
            Assert.That(tObj2!.Value, Is.EqualTo($"failure: tipErr"));

            // AddBusinessUnit success & failure
            var busDto = new busUnitDTO { busUnitId = 1, busUnitName = "Unit" };
            var okRes = new Result { errorCode = 0 };
            MockMvrService.Setup(s => s.AddBusinessUnit(busDto)).Returns(okRes);
            var addRes = TestSubject.AddBusinessUnit(busDto);
            Assert.That(addRes.StatusCode, Is.EqualTo(200));
            Assert.That((addRes as ObjectResult)!.Value, Is.EqualTo(okRes));

            MockMvrService.Setup(s => s.AddBusinessUnit(busDto)).Throws(new Exception("addErr"));
            var addRes2 = TestSubject.AddBusinessUnit(busDto);
            Assert.That(addRes2.StatusCode, Is.EqualTo(200));
            var addObj2 = addRes2 as ObjectResult;
            Assert.That(((Result)addObj2!.Value).errorCode, Is.EqualTo(1));
            Assert.That(((Result)addObj2.Value).errorMessage, Is.EqualTo("addErr"));

            // DeleteBusinessUnit success & failure
            MockMvrService.Setup(s => s.DeleteBusinessUnit(busDto)).Returns(okRes);
            var delRes = TestSubject.DeleteBusinessUnit(busDto);
            Assert.That(delRes.StatusCode, Is.EqualTo(200));
            Assert.That((delRes as ObjectResult)!.Value, Is.EqualTo(okRes));

            MockMvrService.Setup(s => s.DeleteBusinessUnit(busDto)).Throws(new Exception("delErr"));
            var delRes2 = TestSubject.DeleteBusinessUnit(busDto);
            Assert.That(delRes2.StatusCode, Is.EqualTo(200));
            var delObj2 = delRes2 as ObjectResult;
            Assert.That(((Result)delObj2!.Value).errorCode, Is.EqualTo(1));
            Assert.That(((Result)delObj2.Value).errorMessage, Is.EqualTo("delErr"));

            // GetAllProfitCenterListByBU success & failure
            var pcList = new List<ProfitCenterDTO> { new() { profitCenterID = 1 } };
            MockMvrService.Setup(s => s.GetAllProfitCenterListByBU()).Returns(pcList);
            var pcRes = TestSubject.GetAllProfitCenterListByBU();
            Assert.That(pcRes.StatusCode, Is.EqualTo(200));
            Assert.That((pcRes as ObjectResult)!.Value, Is.EqualTo(pcList));

            MockMvrService.Setup(s => s.GetAllProfitCenterListByBU()).Throws(new Exception("pcErr"));
            var pcRes2 = TestSubject.GetAllProfitCenterListByBU();
            Assert.That(pcRes2.StatusCode, Is.EqualTo(200));
            Assert.That(((ObjectResult)pcRes2).Value, Is.EqualTo($"failure: pcErr"));

            // SaveProfitCenter success & failure
            var profitDto = new ProfitCenterDTO { profitCenterID = 1, profitCenter = "P" };
            MockMvrService.Setup(s => s.SaveProfitCenter(profitDto, It.IsAny<UserDto>())).Returns(okRes);
            var saveRes = TestSubject.SaveProfitCenter(profitDto);
            Assert.That(saveRes.StatusCode, Is.EqualTo(200));
            Assert.That(((ObjectResult)saveRes).Value, Is.EqualTo(okRes));

            MockMvrService.Setup(s => s.SaveProfitCenter(profitDto, It.IsAny<UserDto>())).Throws(new Exception("saveErr"));
            var saveRes2 = TestSubject.SaveProfitCenter(profitDto);
            Assert.That(saveRes2.StatusCode, Is.EqualTo(200));
            var saveObj2 = saveRes2 as ObjectResult;
            Assert.That(((Result)saveObj2!.Value).errorCode, Is.EqualTo(1));
            Assert.That(((Result)saveObj2.Value).errorMessage, Is.EqualTo("saveErr"));
        }

        [Test]
        public void GetImportRefDataDTO_ReturnsOrThrows()
        {
            // success
            var list = new List<ImportRefDataColsDefDTO> { new() { importColRecId = 1 } };
            MockMvrService.Setup(s => s.GetImportRefDataDTO()).Returns(list);

            var res = TestSubject.GetImportRefDataDTO();
            Assert.That(res.StatusCode, Is.EqualTo(200));
            Assert.That((res as ObjectResult)!.Value, Is.EqualTo(list));

            // exception path - controller rethrows so test expects exception
            MockMvrService.Setup(s => s.GetImportRefDataDTO()).Throws(new Exception("impErr"));
            Assert.Throws<Exception>(() => TestSubject.GetImportRefDataDTO());
        }

        [Test]
        public void FilterUsageData_HappyAndExceptionPath()
        {
            var usageFilter = new UsageFilterDto();
            var usageList = new List<object> { new { a = 1 } };
            MockMaintCenterService.Setup(m => m.GetUsage(usageFilter)).Returns(usageList);

            var res = TestSubject.FilterUsageData(usageFilter);
            Assert.That(res.StatusCode, Is.EqualTo(200));
            Assert.That((res as ObjectResult)!.Value, Is.EqualTo(usageList));

            // exception path
            MockMaintCenterService.Setup(m => m.GetUsage(usageFilter)).Throws(new Exception("uErr"));
            var res2 = TestSubject.FilterUsageData(usageFilter);
            Assert.That(res2.StatusCode, Is.EqualTo(200));
            Assert.That(((ObjectResult)res2).Value, Is.EqualTo($"failure: uErr"));
        }

        [Test]
        public async Task GeneratePDFReport_ReturnsFileResult_AndHandlesException()
        {
            // Arrange - success
            var form = new FormCollection(new Dictionary<string, Microsoft.Extensions.Primitives.StringValues>
            {
                { "MVRIdString", "1,2,3" },
                { "policyListString", "P" },
                { "businessUnit", "BU" },
                { "showDetailsBool", "false" },
                { "showSummaryBool", "true" }
            });

            var bytes = new byte[] { 1, 2, 3 };
            MockMvrService.Setup(s => s.GeneratePDFReport("BU", It.IsAny<List<string>>(), "P", true, false)).ReturnsAsync(bytes);

            // set Request.Form
            var ctx = TestSubject.ControllerContext.HttpContext;
            ctx.Request.Form = form;

            var result = await TestSubject.GeneratePDFReport();

            Assert.That(result, Is.TypeOf<FileContentResult>());
            var fileResult = result as FileContentResult;
            Assert.That(fileResult!.FileContents, Is.EqualTo(bytes));
            Assert.That(fileResult.FileDownloadName, Is.EqualTo("Motor Vehicle Report.pdf"));
            Assert.That(ctx.Response.Headers.ContainsKey("Content-Disposition"), Is.True);

            // Arrange - exception branch
            MockMvrService.Setup(s => s.GeneratePDFReport("BU", It.IsAny<List<string>>(), "P", true, false))
                          .ThrowsAsync(new Exception("pdfErr"));

            var result2 = await TestSubject.GeneratePDFReport();
            Assert.That(result2, Is.TypeOf<ObjectResult>());
            var obj = result2 as ObjectResult;
            Assert.That(obj!.StatusCode, Is.EqualTo(500));
            Assert.That(((string)obj.Value).StartsWith("Controller: Reports"));
        }
    }
}
eaes error ßa
