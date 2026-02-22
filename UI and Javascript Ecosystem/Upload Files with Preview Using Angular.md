# How to Upload Files with Preview Using Angular (Angular 20 Example)=

In today’s interconnected digital landscape, file uploads have become an essential feature of modern web applications. Whether you’re building a social media platform, an e-commerce website, or a collaborative document management system, allowing users to upload files seamlessly is crucial for a smooth user experience.

Angular has been around since 2010 and remains one of the most popular front-end frameworks. In this article, we will build an **Angular 20 image upload application** where users can:
* Upload an image
* Preview the image before uploading
* View the upload status
* See a list of all uploaded images
* Download an image by clicking on its file name

---

### Technologies Used
* **Angular 20**
* **RxJS**
* **Bootstrap 5**
* **REST API** for Image Upload & Storage



---

### Setting Up the Project

To get started, we’ll create a new Angular project using the Angular CLI.

#### Step 1: Install Angular CLI
```bash
npm install -g @angular/cli@20
```

#### Step 2: Create a New Angular Project
```bash
ng new angular-20-image-upload-preview
cd angular-20-image-upload-preview
```

#### Step 3: Generate Required Components and Services
```bash
ng g s services/file-upload
ng g c components/image-upload
```

---

### Project Structure Explanation
* **app.config.ts**: Used to import and configure required libraries and providers.
* **file-upload.service**: Handles communication with the backend API.
* **image-upload.component**: Contains logic for selecting, previewing, and uploading images.
* **app.component**: The root container for the application.

---

### Set up HttpClient Module
Open `app.config.ts` and import `provideHttpClient` from the Angular Http Module:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

---

### Add Bootstrap 5 to the Angular Project

**Option 1: Use Bootstrap CDN (Quick & Simple)**
Add the following inside the `<head>` tag of your `index.html`:

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
```

**Option 2: Install via npm (Recommended for Production)**
```bash
npm install bootstrap@5
```
Update `angular.json`:
```json
"styles": ["node_modules/bootstrap/dist/css/bootstrap.min.css", "src/styles.css"],
"scripts": ["node_modules/bootstrap/dist/js/bootstrap.bundle.min.js"]
```

---

### Create Angular Service for File Upload
This service uses Angular’s `HttpClient` to send requests to the server.

**services/file-upload.service.ts**
```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class FileUploadService {
  private baseUrl = 'http://localhost:8081';

  constructor(private http: HttpClient) {}

  upload(file: File): Observable<HttpEvent<any>> {
    const formData: FormData = new FormData();
    formData.append('file', file);

    return this.http.post<HttpEvent<any>>(
      `${this.baseUrl}/upload`,
      formData,
      { observe: 'events' }
    );
  }

  getFiles(): Observable<any> {
    return this.http.get(`${this.baseUrl}/files`);
  }
}
```

---

### Create Image Upload Component
This component manages the file selection, the local preview via `FileReader`, and the final upload.

**image-upload.component.ts**
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HttpResponse } from '@angular/common/http';
import { Observable } from 'rxjs';
import { FileUploadService } from '../../services/file-upload.service';

@Component({
  selector: 'app-image-upload',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './image-upload.component.html',
  styleUrl: './image-upload.component.css'
})
export class ImageUploadComponent implements OnInit {
  currentFile?: File;
  preview = '';
  message = '';
  imageInfos?: Observable<any>;

  constructor(private uploadService: FileUploadService) {}

  ngOnInit(): void {
    this.imageInfos = this.uploadService.getFiles();
  }

  selectFile(event: any): void {
    const file = event.target.files?.[0];
    if (file) {
      this.currentFile = file;
      const reader = new FileReader();
      reader.onload = (e: any) => this.preview = e.target.result;
      reader.readAsDataURL(file);
    }
  }

  upload(): void {
    if (!this.currentFile) return;

    this.uploadService.upload(this.currentFile).subscribe({
      next: (event: any) => {
        if (event instanceof HttpResponse) {
          this.message = event.body.message;
          this.imageInfos = this.uploadService.getFiles();
        }
      },
      error: () => {
        this.message = 'Could not upload the image!';
      }
    });
  }
}
```



---

### Image Upload Template (HTML)

```html
<div class="row">
  <div class="col-8">
    <input type="file" accept="image/*" (change)="selectFile($event)" />
  </div>
  <div class="col-4">
    <button class="btn btn-success" [disabled]="!currentFile" (click)="upload()">
      Upload
    </button>
  </div>
</div>

<div *ngIf="preview">
  <img [src]="preview" class="preview mt-3" style="max-width: 100%; height: auto;" />
</div>

<div *ngIf="message" class="alert alert-info mt-3">
  {{ message }}
</div>

<div class="card mt-3">
  <div class="card-header">Uploaded Images</div>
  <ul class="list-group list-group-flush">
    <li class="list-group-item" *ngFor="let image of imageInfos | async">
      <a [href]="image.url" target="_blank">{{ image.name }}</a>
    </li>
  </ul>
</div>
```

---

### Run the Application
Start the Angular app:
```bash
ng serve --port 8083
```
Open your browser at `http://localhost:8083/` to see your functional image upload tool with real-time preview!

Would you like me to find a **sample backend implementation in Node.js or Spring Boot** that pairs with this Angular component to handle the actual file storage?
```markdown
Would you like me to find a **sample backend implementation in Node.js or Spring Boot** that pairs with this Angular component to handle the actual file storage?
```