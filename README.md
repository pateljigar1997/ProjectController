# ProjectController


using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using PROJECT.Models;
using PROJECT.ViewModels;
using ReflectionIT.Mvc.Paging;

namespace PROJECT.Controllers
{
    public class HomeController : Controller
    {
        
        private readonly ICreateRepository createRepository;
        private readonly IHostingEnvironment hostingEnvironment;
        

        public HomeController(ICreateRepository createRepository, IHostingEnvironment hostingEnvironment)
        {
            this.createRepository = createRepository;
            this.hostingEnvironment = hostingEnvironment;
            
        }

        [HttpGet]
        public IActionResult Create()
        {
            return View();
        }

        private string ProcessUploadedFile(CreateViewModel model)
        {
            string uniqueFileName = null;
            if (model.PhotoPath != null && model.PhotoPath.Count > 0)
            {
                foreach (IFormFile photo in model.PhotoPath)
                {
                    string uploadsFolder = Path.Combine(hostingEnvironment.WebRootPath, "images");
                    uniqueFileName = Guid.NewGuid().ToString() + "_" + photo.FileName;
                    string filepath = Path.Combine(uploadsFolder, uniqueFileName);
                    using (var fileStream = new FileStream(filepath, FileMode.Create))
                    {
                        photo.CopyTo(fileStream);
                    }

                }
            }

            return uniqueFileName;
        }


        [HttpPost]
        public IActionResult Create(CreateViewModel model)
        {
            if (ModelState.IsValid)
            {
                string uniquefilename = ProcessUploadedFile(model);
                Create newCreate = new Create
                {
                    Name = model.Name,
                    Email = model.Email,
                    PhoneNumber = model.PhoneNumber,
                    Gender = model.Gender,
                    PhotoPath = uniquefilename
                };
                createRepository.Add(newCreate);
                return RedirectToAction("Index");
            }
            return View();
        }

        public IActionResult Index()
        {
            //ViewData["CurrentFilter"] = searchString;
            ////var member = createRepository.GetAllRegister();
            ////if (!string.IsNullOrEmpty(search))
            ////{
            ////    var member = createRepository.SearchMember(search);
            ////    //model.Customers = 
            ////    // = qry.Where(p => p.Name.Contains(search));
            ////    return View(member);
            ////}
            // var model = await PagingList.CreateAsync(qry,10,page)
            //var model = new Paging(createRepository);
            //var x = model.OnGetAsync();
            var items = createRepository.GetAllRegister();
            //var model = await PagingList<Create>.CreateAsync(items, 5, page);
            //model.RouteValue = new Microsoft.AspNetCore.Routing.RouteValueDictionary {
            //  { "search",search}
            //};
            //return View(model);
            return View(items);
        }
        [HttpPost]
        public ActionResult Index(string searchtext)
        {
            var users = createRepository.GetAllRegister();
            if (searchtext != null)
            {
              createRepository.Search(searchtext); 
            }
            return View(users);
        }
        [HttpGet]
        public ViewResult Edit(int id)
        {
            Create employee = createRepository.GetMember(id);
            EditViewModel employeeEditViewModel = new EditViewModel
            {
                Id = employee.Id,
                Name = employee.Name,
                Email = employee.Email,
                Status = employee.Status,
                ExistingPhoto = employee.PhotoPath
            };
            return View(employeeEditViewModel);
        }

        [HttpPost]
        public IActionResult Edit(EditViewModel model)
        {
            // Check if the provided data is valid, if not rerender the edit view
            // so the user can correct and resubmit the edit form
            if (ModelState.IsValid)
            {
                // Retrieve the employee being edited from the database
                Create employee = createRepository.GetMember(model.Id);
                // Update the employee object with the data in the model object
                employee.Name = model.Name;
                employee.Email = model.Email;
                employee.Status = model.Status;


                if (model.PhotoPath != null)
                {

                    if (model.ExistingPhoto != null)
                    {
                        string filePath = Path.Combine(hostingEnvironment.WebRootPath,
                            "images", model.ExistingPhoto);
                        System.IO.File.Delete(filePath);
                    }

                    employee.PhotoPath = ProcessUploadedFile(model);
                }


                createRepository.Edit(employee);

                return RedirectToAction("index");
            }

            return View(model);
        }

        //For Chekbox
        [HttpPost]
        public ActionResult ConfirmDelete1(string ids)
         {
            if (ids != null)
            {
                var x = ids.Split(",");
                for (int i = 0; i < x.Count(); i++)
                {
                    if(x[i] != null) { 
                    int UID = Convert.ToInt32(x[i]);
                    createRepository.Delete(UID);
                    createRepository.Save();
                    }
                }
            }
            return RedirectToAction("Index", "Home");
        }

        [HttpPost]
        [ActionName("Delete")]
        public ActionResult ConfirmDelete(int id)
        {
            createRepository.Delete(id);
            createRepository.Save();

            return RedirectToAction("Index", "Home");
        }

        [HttpGet]
        public ActionResult Delete(int id)
        {
            var create = createRepository.GetMember(id);
            return View(create);
        }

        [HttpPost]
        public ActionResult ChangeStatus(string sids)
        {
            if (sids != null)
            {
                var x = sids.Split(",");
                for (int i = 0;i< x.Count(); i++)
                {
                    var UID = Convert.ToInt32(x[i]);
                    Create employee = createRepository.GetMember(UID);
                    employee.Status = !(employee.Status);
                    createRepository.Edit(employee);
                }
            }
            return RedirectToAction("index", "home");
        }
    }
}
