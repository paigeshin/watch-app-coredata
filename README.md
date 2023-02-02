# watch-app-coredata

```swift

import CoreData

struct Persistentcontroller {
    
    static let shared = Persistentcontroller()
    
    let container: NSPersistentContainer
    
    init(inMemory: Bool = false) {
        self.container = NSPersistentContainer(name: "Task")
        if inMemory {
            self.container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        self.container.loadPersistentStores { desc, error in
            if let error: Error {
                print(error)
            }
        }
    }
    
}


```

```swift
@main
struct WatchApp_Watch_AppApp: App {
    
    let container = Persistentcontroller.shared.container
    
    var body: some Scene {
        WindowGroup {
            NavigationView {
                ContentView()
            }
            .environment(\.managedObjectContext, self.container.viewContext)
        }
    }
}

```

```swift
import SwiftUI
import CoreData

struct ContentView: View {
    var body: some View {
        Home()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct Home: View {
    
    var body: some View {
        // Geometry Reader For Getting Frame...
        GeometryReader { reader in
            
            let rect = reader.frame(in: .global)
            
            VStack(spacing: 15) {
                HStack(spacing: 25) {
                    
                    NavigationLink(destination: AddItem()) {
                        NavButton(image: "plus", title: "Memo", rect: rect, color: .pink)
                    }
                    .buttonStyle(.plain)
                    
                    NavigationLink(destination: DeleteMemo(),
                                   label: {
                        NavButton(image: "trash", title: "delete", rect: rect, color: .red)
                    })
                    .buttonStyle(.plain)
                    
                }
                .frame(width: rect.width, alignment: .center)
                
                HStack(spacing: 25) {
                    
                    NavigationLink(destination: ViewMemo()) {
                        NavButton(image: "doc.plaintext", title: "Memo", rect: rect, color: .blue)
                    }
                    .buttonStyle(.plain)
                    
                    NavButton(image: "star", title: "Rating", rect: rect, color: .orange)
                    
                }
                .frame(width: rect.width, alignment: .center)
            } //: VSTACK
        } // GEO
    }
    
}

struct AddItem: View {
    
    @State var memoText = ""
    
    // getting context from envrionment...
    @Environment(\.managedObjectContext) var context
    @Environment(\.dismiss) var dismiss
    
    // Edit Option
    var memoItem: Memo?
    
    var body: some View {
        VStack(spacing: 15) {
            
            TextField("Memories...", text: $memoText)
            
            Button {
                addMemo()
            } label: {
                Text("Save")
                    .padding(.vertical, 10)
                    .frame(maxWidth: .infinity, alignment: .center)
                    .background(Color.orange)
                    .cornerRadius(15)
            }
            .padding(.horizontal)
            .buttonStyle(.plain)
            .disabled(memoText == "")
            
        }
        .navigationTitle("Add Memo")
        .onAppear {
            
            // Verifying it memo item has data...
            if let memoItem {
                memoText = memoItem.title ?? ""
            }
            
        }
    }
    
    func addMemo() {
        let memo = memoItem == nil ? Memo(context: context) : memoItem!
        memo.title = memoText
        memo.dateAdded = Date()
        try? context.save()
        dismiss()
    }
    
}

struct ViewMemo: View {
    
    @FetchRequest(entity: Memo.entity(), sortDescriptors: [NSSortDescriptor(keyPath: \Memo.dateAdded, ascending: false)], animation: .easeIn) var results: FetchedResults<Memo>
    
    var body: some View {
        List(results) { item in
            HStack(spacing: 10) {
                VStack(alignment: .leading, spacing: 3) {
                    Text(item.title ?? "")
                        .font(.system(size: 12))
                        .foregroundColor(.white)
                    
                    Text("Last Modified")
                        .font(.system(size: 8))
                        .fontWeight(.medium)
                        .foregroundColor(.gray)
                    
                    Text(item.dateAdded ?? Date(), style: .date)
                        .font(.system(size: 8))
                        .fontWeight(.bold)
                        .foregroundColor(.gray)
                }
                
                Spacer(minLength: 4)
                
                NavigationLink(destination: AddItem(memoItem: item), label: {
                    Image(systemName: "square.and.pencil")
                        .font(.callout)
                        .foregroundColor(.white)
                        .padding(8)
                        .backgroundStyle(Color.orange)
                        .cornerRadius(8)
                })
                .buttonStyle(.plain)
                
            }
        }
        .listStyle(CarouselListStyle())
        .padding(.top)
        .overlay(
            Text(results.isEmpty ? "No Memo's found" : "")
        )
        .navigationTitle("Memo's")
    }
    
}

struct DeleteMemo: View {
    
    @FetchRequest(entity: Memo.entity(), sortDescriptors: [NSSortDescriptor(keyPath: \Memo.dateAdded, ascending: false)], animation: .easeIn) var results: FetchedResults<Memo>
    
    @State var deleteMemoItem: Memo?
    @State var deleteMemo = false
    
    // context
    @Environment(\.managedObjectContext) var context
    
    var body: some View {
        List(results) { item in
            HStack(spacing: 10) {
                VStack(alignment: .leading, spacing: 6) {
                    Text(item.title ?? "")
                        .font(.system(size: 12))
                        .foregroundColor(.white)
                    
                    Text(item.dateAdded ?? Date(), style: .date)
                        .font(.system(size: 8))
                        .fontWeight(.bold)
                        .foregroundColor(.gray)
                }
                
                Spacer(minLength: 4)
                
                Button {
                    
                    deleteMemoItem = item
                    deleteMemo.toggle()
                    
                } label: {
                    Image(systemName: "trash")
                        .font(.callout)
                        .foregroundColor(.white)
                        .padding(8)
                        .background(Color.red)
                        .cornerRadius(8)
                }
                .buttonStyle(.plain)

                
            }
        }
        .listStyle(CarouselListStyle())
        .padding(.top)
        .overlay(
            Text(results.isEmpty ? "No Memo's to delete" : "")
        )
        .navigationTitle("Delete Memo")
        .alert(isPresented: $deleteMemo) {
            Alert(title: .init("Confirm"), message: .init("To delete this memo"), primaryButton: .default(Text("Ok"), action: {
                deleteMemo(memo: deleteMemoItem!)
            }), secondaryButton: .destructive(.init("Cancel")))
        }
    }
    
    // Delete Memo...
    
    func deleteMemo(memo: Memo) {
        context.delete(memo)
        try? context.save()
    }
    
}

struct NavButton: View {
    
    var image: String
    var title: String
    var rect: CGRect
    var color: Color
    
    var body: some View {
        
        VStack(spacing: 8) {
            Image(systemName: image)
                .font(.title2)
                .frame(width: rect.width / 3, height: rect.width / 3, alignment: .center)
                .background(color)
                .clipShape(Circle())
            
            Text(title)
                .font(.system(size: 10))
                .foregroundColor(.white)
        }
        
    }
    
}


```
