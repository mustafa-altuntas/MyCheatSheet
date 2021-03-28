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