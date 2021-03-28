
# A) Projenin oluşturulması
## 1) Önce bir asp .net core web mvc uygulaması oluşturuyoruz.
## 2) Projemizde kullanacağımız kütüphaneleri yüklüyoruz
* ``Microsoft.AspNetCore.Identity.EntityFrameworkCore``
* ``Microsoft.EntityFrameworkCore``
* ``Microsoft.EntityFrameworkCore.SqlServer``
* ``Microsoft.EntityFrameworkCore.Tools``
* ``AutoMapper.Extensions.Microsoft.DependencyInjection``

## 3) Startup.cs dosyasında ilk kurulum ayarlarıunı yapıyoruz.
```c#
public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {

            services.AddMvc(option => option.EnableEndpointRouting = false);

        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {

            app.UseDeveloperExceptionPage();
            app.UseStatusCodePages();
            app.UseStaticFiles();
            app.UseMvcWithDefaultRoute();
            app.UseAuthentication();
            app.UseStaticFiles();

        }
    }
```

# B) Identity yapısına uygun dbContext ve class'ları oluşturma
## 1) AppUser.cs class'ının oluşturulması
AppUser class'ı kullanıcıları tuttuğumuz tablomuz. Models klasöründe AppUser adında class oluşturalım ve class'ımıza ``IdentityUser``'ı uygulayalım.
```c#
    public class AppUser:IdentityUser
    {
    }
```
``IdentityUser``'ı uyguladığmız için ``Identity`` mimarisinde user için gerekli tüm propertileri ``IdentityUser`` dan çekecek.
## 2) AppIdentityDbContext class'ı oluşturma
Burda ``Identity`` servisimizin tabloları için herhangi bir ``DbSet`` tanılamamız gerekmiyor. Eğer ``Identity`` mimarisinin dışıda kendi özel tablolarım varsa bunlar için ``DbSet`` oluşturmak zorundayız.

````c#
    public class AppIdentityDbContext:IdentityDbContext<AppUser>
    {
        public AppIdentityDbContext(DbContextOptions<AppIdentityDbContext> options):base(options)
        {

        }
    }
````
## 3) Startup.cs dosyasının ayarlanması
Tüm bunlardan sonra artık ``Startup.cs`` dasyamızda ``db`` ve ``Identity`` bağlantılarını yapabiliriz.
```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddAutoMapper(typeof(Startup));

    services.AddDbContext<AppIdentityDbContext>(options =>
    {
        options.UseSqlServer(Configuration["ConnectionStrings:DefaultConnectionString"]);
    });

    services.AddIdentity<AppUser, IdentityRole>()
        .AddEntityFrameworkStores<AppIdentityDbContext>();

    services.AddMvc(option => option.EnableEndpointRouting = false);

}
```

# C) Identity ile gelen hazır methotlar