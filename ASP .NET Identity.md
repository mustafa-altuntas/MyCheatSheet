# ASP .NET IDENTITY İle örnek bir proje geliştirerek Identity konusunu inceleyeceğiz


# SingUp kullanıcı kaydı
Kullanıcı kaydını ``userManager.CreateAsync()`` methodu ile yapıyoruz.
``CreateAsync()`` methodu iki parametre alıyor bunlar user modeli ve şifre burada aldığı şifreyi kendi otomatik hash'liyor.
``CreateAsync()`` methodu geriye IdentityError dönüyor bizde bu sayede kullanıcının kaydı esnasında herhangi bir hata oluşursa bu hataları görüntüleyebiliyoruz. Örneğin bu kullanıcı adı zaten mevcut, şifrenizde büyük küçük harf olmak zorunda gibi...

Aşağıdaki kod örneğinde ``AppUser`` bizim Identity veritabanımızdaki ``User`` tablosuna karşılık geliyor.

```c#

        private UserManager<AppUser> userManager { get; }
        private readonly IMapper _mapper;

public IActionResult SingUp()
        {
            return View();
        }

        [HttpPost]
        public async Task<IActionResult> SingUp(UserViewModel vm)
        {

            if (ModelState.IsValid)
            {
                var model = _mapper.Map<AppUser>(vm);
                IdentityResult result =  await userManager.CreateAsync(model, vm.Password);
                if (result.Succeeded)
                {
                    return RedirectToAction("LogIn");
                }
                else
                {
                    foreach (IdentityError item in result.Errors)
                    {
                        ModelState.AddModelError("", item.Description);
                    }
                }   
            }
            return View(vm);
        }
```

# Şifre Doğrulama Ayarları Password Validasyon

Bu ayarlar StartUp.cs de yapılacaktır.

Bu validasyonlar işre için varsayılan ayarlardır.
|||
|---|---|
|RequiredLength |Mutlaka uzunluğu n kadar olamalıdır|
|RequireNonAlphanumeric|Mutlaka bir alfanumerik olmayan karakter olmalıdır |
|RequireLowercase|Mutlaka bir küçük harf olmalıdır |
|RequireUppercase|Mutlaka bir büyük harf olmalıdır |
|RequireDigit| Mutlaka bir sayısal karakter olmalıdır|


```c#
services.AddIdentity<AppUser, AppRole>(opts=> 
{
    opts.Password.RequiredLength = 4;
    opts.Password.RequireNonAlphanumeric = false;
    opts.Password.RequireLowercase = false;
    opts.Password.RequireUppercase = false;
    opts.Password.RequireDigit = false;
}).AddEntityFrameworkStores<AppIdentityDbContext>();
```

## # Custom Şifre Doğrulama Mekanizması (Custom Password Validatior)
- `IPasswordValidator<AppUser>`

Kendi şifre validasyonumuzu yazmak için projemizde :open_file_folder:``CustomValidation`` adında bir klasör oluşturduk ve bu klasörün içerisine ``CustomePasswordValidator.cs`` class'ını oluşturduk ve bu class 'ımızı ``IPasswoed.validator<AppUser>`` 'ı implamenta ettik.


```c#
public class CustomePasswordValidator : IPasswordValidator<AppUser>
    {
        public Task<IdentityResult> ValidateAsync(UserManager<AppUser> manager, AppUser user, string password)
        {

            List<IdentityError> errors = new List<IdentityError>();

            if (password.ToLower().Contains(user.UserName.ToLower()))
            {
                errors.Add(new IdentityError() { Code = "SifreKullanıcıAdıIceriyor", Description="Şifreniz Kullanıcı Adınızı içeremez." });
            }


            if (password.ToLower().Contains("1234"))
            {
                errors.Add(new IdentityError() { Code = "Sifre1234Iceriyor", Description = "Şifreniz 1234 sayısını içeremez." });
            }

            if (errors.Count ==0)
            {
                return Task.FromResult(IdentityResult.Success);
            }
            else
            {
                return Task.FromResult(IdentityResult.Failed(errors.ToArray()));
            }
        }
    }
```
CustomePasswordValidator.cs class'ımızı oluşturduk ve şimdi sıra bu class'ı StartUp.cs de ``AddPasswordValidator()`` methodu ile kullanmada.

```c#
    services.AddIdentity<AppUser, AppRole>(opts =>
    {
        opts.Password.RequiredLength = 4;
        opts.Password.RequireNonAlphanumeric = false;
        opts.Password.RequireLowercase = false;
        opts.Password.RequireUppercase = false;
        opts.Password.RequireDigit = false;
    }).AddPasswordValidator<CustomePasswordValidator>() //burası
        .AddEntityFrameworkStores<AppIdentityDbContext>();
```

# Kullanıcı Doğrulama Ayarları (User Validation)
UserName alanı benzersiz olmalı ve bu ayar kapaılamaz.
## # Default
- RequireUniqueEmail -> Email adresinin unique olup olmadığı
- AllowedUserNameCharacters -> UserNama için hangi karakterlere izin verilecek


```c#
services.AddIdentity<AppUser, AppRole>(opts =>
{
    /////////////////////////////////////////////////////////////
    opts.User.RequireUniqueEmail = true;
    opts.User.AllowedUserNameCharacters = "abcçdefğghiıjklmnoöpqrsştüuvwxyzABCÇİĞŞÜDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._";
    ////////////////////////////////////////////////////////////


    opts.Password.RequiredLength = 4;
    opts.Password.RequireNonAlphanumeric = false;
    opts.Password.RequireLowercase = false;
    opts.Password.RequireUppercase = false;
    opts.Password.RequireDigit = false;
}).AddPasswordValidator<CustomePasswordValidator>()
    .AddEntityFrameworkStores<AppIdentityDbContext>();
```

## # Custom Kullanıcı Doğrulama Mekanizması (Custom User Validatior)
- ``IUserValidator<AppUser>``


```c#
public class CustomUserValidator : IUserValidator<AppUser>
    {
        public Task<IdentityResult> ValidateAsync(UserManager<AppUser> manager, AppUser user)
        {
            List<IdentityError> errors = new List<IdentityError>();
            string[] Digits = new string[] { "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" };

            foreach (var digit in Digits)
            {
                if (user.UserName[0].ToString() == digit)
                {
                    errors.Add(new IdentityError() { Code = "KullaniciAdiIlkKarakterSayisal", Description = "Kullanıcı Adınız bir sayı ile başlayamaz." });
                }
            }

            if (errors.Count == 0)
            {
                return Task.FromResult(IdentityResult.Success);
            }
            else
            {
                return Task.FromResult(IdentityResult.Failed(errors.ToArray()));
            }

        }
    }
```

```c#
services.AddIdentity<AppUser, AppRole>(opts =>
{
    opts.User.RequireUniqueEmail = true;
    opts.User.AllowedUserNameCharacters = "abcçdefğghiıjklmnoöpqrsştüuvwxyzABCÇİĞŞÜDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._";

    opts.Password.RequiredLength = 4;
    opts.Password.RequireNonAlphanumeric = false;
    opts.Password.RequireLowercase = false;
    opts.Password.RequireUppercase = false;
    opts.Password.RequireDigit = false;
}).AddPasswordValidator<CustomePasswordValidator>()
  .AddUserValidator<CustomUserValidator>() // burası ekledi
  .AddEntityFrameworkStores<AppIdentityDbContext>();
```

## Doğrulama Mesajlarının Türkçeleştirilmesi
* IdentityErrorDescriber
  * InvalidUserName
  * DuplicateEmail
  * ve bir çok override edilebilir method içerir

```c#
public class CustomIdentityErrorDescriber:IdentityErrorDescriber
    {

        public override IdentityError InvalidUserName(string userName)
        {
            return new IdentityError() { Code = "InvalidUserName", Description = $"Bu {userName} ismi geçersizdir." };
        }

        public override IdentityError DuplicateUserName(string userName)
        {
            return new IdentityError() { Code = "DuplicateUserName", Description = $"Bu {userName} ismi kullanılmaktadır." };
        }

        public override IdentityError DuplicateEmail(string email)
        {
            return new IdentityError() { Code = "DuplicateEmail", Description = $"Bu {email} email adresi kullanılmaktadır." };

        }

        public override IdentityError InvalidEmail(string email)
        {
            return new IdentityError() { Code = "InvalidEmail", Description = $"Bu {email} email adresi geçersizdir." };
        }

        public override IdentityError PasswordTooShort(int length)
        {
            return new IdentityError() { Code = "PasswordTooShort", Description = $"Şifreniz en az {length} karakterli olmalıdır." };
        }
    }
```

```c#
services.AddIdentity<AppUser, AppRole>(opts =>
{
    opts.User.RequireUniqueEmail = true;
    opts.User.AllowedUserNameCharacters = "abcçdefğghiıjklmnoöpqrsştüuvwxyzABCÇİĞŞÜDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._";

    opts.Password.RequiredLength = 4;
    opts.Password.RequireNonAlphanumeric = false;
    opts.Password.RequireLowercase = false;
    opts.Password.RequireUppercase = false;
    opts.Password.RequireDigit = false;
}).AddPasswordValidator<CustomePasswordValidator>()
  .AddUserValidator<CustomUserValidator>()
  .AddErrorDescriber<CustomIdentityErrorDescriber>()  /// eklendi
  .AddEntityFrameworkStores<AppIdentityDbContext>();
```

