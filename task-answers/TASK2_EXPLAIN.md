# Task 2

Courier có thể đăng ký tài khoản nhưng không được hoạt động ngay
Courier phải được Admin approve trước khi nhận delivery orders.

Courier status enum:

PENDING || APPROVED || REJECTED

Chỉ courier APPROVED mới có thể hoạt động.

---

# Business Flow

1. Courier gửi request đăng ký
2. System tạo courier với status = PENDING
3. Admin vào trang quản lý courier
4. Admin có thể:
   - APPROVE courier
   - REJECT courier (phải nhập reason)
5. Courier APPROVED mới được nhận delivery orders

---

# Database

## Courier Model

```csharp
public class Courier
{
    public int Id { get; set; }

    public string Name { get; set; }

    public string Phone { get; set; }

    public string Email { get; set; }

    public CourierStatus Status { get; set; } = CourierStatus.PENDING;

    public string? RejectionReason { get; set; }

    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

Enum:

```csharp
public enum CourierStatus
{
    PENDING,
    APPROVED,
    REJECTED
}
```

---

# DTO

```csharp
public class CreateCourierDto
{
    [Required]
    public string Name { get; set; }

    [Required]
    public string Phone { get; set; }

    [Required]
    public string Email { get; set; }
}
```

```csharp
public class UpdateStatusDto
{
    public CourierStatus Status { get; set; }

    public string? Reason { get; set; }
}
```

---

# Courier Service

```csharp
public class CourierService
{
    private readonly AppDbContext _context;

    public CourierService(AppDbContext context)
    {
        _context = context;
    }

    public Courier Register(CreateCourierDto dto)
    {
        var courier = new Courier
        {
            Name = dto.Name,
            Phone = dto.Phone,
            Email = dto.Email,
            Status = CourierStatus.PENDING
        }; //chỗ này sẽ dùng IMapper để map sau, hiện tại map tay

        _context.Couriers.Add(courier);
        _context.SaveChanges();

        return courier;
    }

    public Courier UpdateStatus(int id, UpdateStatusDto dto)
    {
        var courier = _context.Couriers.Find(id);

        if (courier == null)
            throw new Exception("Courier not found");

        if (dto.Status == CourierStatus.REJECTED && string.IsNullOrEmpty(dto.Reason))
            throw new Exception("Reject reason required");

        courier.Status = dto.Status;
        courier.RejectionReason = dto.Reason;

        _context.SaveChanges();

        return courier;
    }
}
```

---

# Courier Controller

```csharp
[ApiController]
[Route("couriers")]
public class CouriersController : ControllerBase
{
    private readonly CourierService _service;

    public CouriersController(CourierService service)
    {
        _service = service;
    }

    // Đăng ký Courier
    [HttpPost("register")]
    public IActionResult Register([FromBody] CreateCourierDto dto)
    {
        var courier = _service.Register(dto);
        return Ok(courier);
    }

    // Admin duyệt hoặc từ chối một courier
    [Authorize(Roles = "Admin")]
    [HttpPost("{id}/status")]
    public IActionResult UpdateStatus(int id, [FromBody] UpdateStatusDto dto)
    {
        var courier = _service.UpdateStatus(id, dto);
        return Ok(courier);
    }

    // CRUD
    [HttpGet]
    public IActionResult GetCouriers()
    {
        return Ok(_service.GetAll());
    }

    [HttpGet("{id}")]
    public IActionResult GetCourier(int id)
    {
        return Ok(_service.GetById(id));
    }
}
```

---

# Đăng ký Courier theo OTP

Flow chạy theo các bước:

```

POST /couriers/send-otp
POST /couriers/verify-otp
POST /couriers/register

```

Các bước:

1. Courier nhập số điện thoại hoặc email
2. Hệ thống gửi OTP về
3. Courier nhập OTP để xác nhận
4. Xác nhận xong thì hệ thống mới tạo tài khoản courier với status = PENDING

Đăng ký xong chưa được chạy đơn liền, phải chờ admin duyệt trước

---

# Admin duyệt hoặc từ chối Courier

Endpoint:

```

POST /couriers/{id}/status

```

Approve:

```

{
"status": "APPROVED"
}

```

Reject:

```

{
"status": "REJECTED",
"reason": "Invalid documents"
}

```

- Nếu admin duyệt thì status chuyển thành APPROVED
- Nếu admin từ chối thì status chuyển thành REJECTED
- Trường hợp reject thì bắt buộc phải có lý do, không được để trống.

---

# Xác thực và Phân quyền

Xác thực:

- Các field bắt buộc trong DTO phải có
- Nếu status là REJECTED thì phải có reason

Phân quyền:

```

Admin only

```

- API duyệt courier chỉ cho admin gọi
- Courier thường không được gọi API này

---

# Test cho flow duyệt courier

Ví dụ test case:

```

Test: Admin duyệt courier

```

Các bước test:

1. Tạo một courier mới với status = PENDING
2. Gọi API approve courier
3. Kiểm tra lại status phải đổi thành APPROVED

Nếu status đổi đúng thì flow duyệt courier hoạt động bình thường.
