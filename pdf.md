# Lightpack PDF Service

A robust, extensible PDF generation API for Lightpack applications, featuring driver-based architecture, seamless storage integration, and full HTTP response support.

## Features
- **Driver-based architecture** (Dompdf supported, extensible)
- **Render HTML or templates to PDF**
- **Set metadata** (title, author, subject, keywords)
- **Add pages, embed images, Unicode support**
- **Output as download, stream, or save to storage**
- **Advanced driver access for custom features**

## Quick Start

### 1. Install Dompdf

```
composer require dompdf/dompdf
```

### 2. Basic Usage

```php
$pdf = app('pdf');
$pdf->setTitle('Invoice')
    ->setAuthor('Lightpack')
    ->html('<h1>Invoice #123</h1>');

// Download
$pdf->download('invoice.pdf');

// Stream inline
$pdf->stream('invoice.pdf');

// Save to configured storage
$pdf->save('invoices/invoice-123.pdf');
```

## API Reference

### Set Metadata
```php
$pdf->setMeta([
    'title' => 'Report',
    'author' => 'Admin',
    'subject' => 'Monthly',
    'keywords' => 'report, monthly, pdf',
]);
```

### Render from HTML string or view template
```php
$pdf->html('<h1>Hello</h1>');
$pdf->template('invoice', ['order' => $order]);
```

### Add Pages
```php
$pdf->addPage();
```

### Download as Attachment
```php
return $pdf->download('myfile.pdf');
```

### Stream Inline
```php
return $pdf->stream('myfile.pdf');
```

### Save to Storage
```php
$pdf->save('reports/2025-05-15/report.pdf');
```

### Advanced: Access Driver Instance
```php
$driver = $pdf->getDriver();
$dompdf = $driver->getInstance();
$dompdf->setPaper('A4', 'landscape');
```

## Storage Integration
- The `save()` method uses the Lightpack `storage` service.
  
```php
$pdf->save('public/invoices/invoice-123.pdf');
$url = app('storage')->url('public/invoices/invoice-123.pdf');
```

---